---
layout: post
title:  "Some iOS Table View Patterns"
date:   2016-11-10 12:00:00 -0400
categories: swift, ios
---

I plan on making a series of posts regarding some patterns I like to use while writing iOS apps ranging from `UITableViews` and their delegates/datasources, reuse of classes using generics, and navigation flow. Today, I'll start with `UITableViews.`

#### UITableView ####

Let's start with a basic view controller. I'll write everything out using just code for clarities sake.

{% highlight swift %}
class MyTableViewController: UIViewController {
  let tableView = UITableView(frame: CGRect.zero, style: .plain)
}
{% endhighlight %}

I like to abstract out the `UITableViewDataSource` and `UITableViewDelegate` into a seperate class for reusability's and avoiding code repetition.

{% highlight swift %}
class DataSource<Provider: DataProvider, Cell: UITableViewCell>: NSObject, UITableViewDataSource, UITableViewDelegate where Cell: ConfigurableCell, Provider.Object == Cell.Object {
  // will fill out shortly
}
{% endhighlight %}

First, you'll notice that it's doubly generic over two types: `Provider` which is of the type `DataProvider` and `Cell` which is a simple `UITableViewCell` which conforms to `ConfigurableCell` (from `where Cell: ConfigurableCell`). I'll go over `ConfigurableCell` first.

{% highlight swift %}
protocol ConfigurableCell {
  associatedtype Object
  static var identifier: String { get }
  func configure(for object: Object, at indexPath: IndexPath)
}
class MyTableViewCell: UITableViewCell { //customize it here }
extension MyTableViewCell: ConfigurableCell {
  func configure(for object: String, at indexpath: IndexPath) {
    textLabel?.text = object
    // some other configuration
  }
}
{% endhighlight %}

The protocol `ConfigurableCell` is quite simple, it just states that the cell that conforms to it must implement a function which is used to configure the cell for it's display. This function will be the sole function called in `cellForRowAtIndexPath`. The `associatedtype` is a placeholder type for which the cell is designed to be configured to. So, for example, if the data you want to display in your tableView is an `Array` of `Book`s, your `configure` function would be

{% highlight swift %}
struct Book {
  let title: String
  let author: String
}
func configure(for object: Book, at indexPath: IndexPath) {
  textLabel?.text = object.title
  detailTextLabel?.text = "By: " + object.author
}
{% endhighlight %}

Next, we have the `DataProvider` type. This is quite simple. This is just a protocol the holder of the objects you want to display conform to.

{% highlight swift %}
protocol DataProvider {
  associatedtype Object
  func numberOfSections() -> Int
  func numberOfRows(inSection: Int) -> Int
  func object(at indexPath: IndexPath) -> Object
}
{% endhighlight %}

The purpose of this class is obviously to fill out the reqruied `UITableViewDataSource` methods. Further extension of this protocol can provide further functionality such as insertion/removal/append/etc. But we'll keep it simple here.

A typical type I make is `ArrayProvider`:

{% highlight swift %}
class ArrayProvider<Type>: DataProvider {
  var objects: [Type]
  func numberOfSections() -> Int { return 1}
  func numberOfRows(inSection section: Int) -> Int { return objects.count }
  func object(at indexPath: IndexPath) -> Object { return objects[indexPath.row] }
}
{% endhighlight %}

Now, we move back to the `DataSource`:

{% highlight swift %}
class DataSource<Provider: DataProvider, Cell: UITableViewCell>: NSObject, UITableViewDataSource, UITableViewDelegate where Cell: ConfigurableCell, Provider.Object == Cell.Object {

  let provider: Provider
  let tableView: UITableView

  init(tableView: UITableView, provider: Provider) {
    self.tableView = tableView
    self.provider = provider
    super.init()

    setup()
  }

  func setup() {
    tableView.dataSource = self
    tableView.delegate = self
    tableView.register(Cell.self, forReuseIdentifier: Cell.identifier)
  }

  func numberOfSections(in tableView: UITableView) -> Int {
    return provider.numberOfSections()
  }
  func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
    return provider.numberOfItems(in: section)
  }
  func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    let cell = tableView.dequeueReusableCell(withIdentifier: Cell.identifier, for: indexPath) as! Cell
    let object = provider.object(at: indexPath)
    cell.configure(for: object, at: indexPath)
    return cell
  }
}
{% endhighlight %}

Now, the purpose of the `DataSource` becomes rather clear. You feed it a `UITableView` and a `DataProvider` and it handles all the connections. It's reusable and isolated from the rest of your code. Any further modification can be handled in a variety of ways, but the repetitive boiler plate is isolated and regularized.

A useful subclass I create is a `DataProvider` tailored to the aforementioned `ArrayProvider`.

{% highlight swift %}
class ArrayDataSource<Type,Cell: UITableViewCell>: DataSource<ArrayProvider<Type>,Cell> where Cell: ConfigurableCell, Cell.Object == Type {
    init(tableView: UITableView, array: [Type]) {
        let provider = ArrayProvider(array: array)
        super.init(tableView: tableView, provider: provider)
    }

    var objects: [Type] { return provider.objects }
}
{% endhighlight %}

This subclass simply provides an initializer for when your objects are contained in an array so you can use the `ArrayProvider` type. To be honest, I'd prefer this be a part of the original `DataSource` class, but Swift's type system doesn't allow recursive constraints, yet, and that would be a requirement to write an extension where the `where` clauses imply recursion. So I simply subclass, for now. (This is a problem I'm running into exceedingly often the more and more I try to use generic code.)

Now, we can finally set up our view controller.

{% highlight swift %}
class MyTableViewController: UIViewController {

  let tableView = UITableView(frame: CGRect.zero, style: .plain)
  var dataSource: ArrayDataSource<Book,MyTableViewCell>!


  override func viewDidLoad() {
    super.viewDidLoad()

    var books = [("The Hobbit","Tolkien"),("Harry Potter","Rowling")].map { book in Book(title: book.0, author: book.1) }
    dataSource = ArrayDataSource(tableView: tableView, array: books)
  }
}
{% endhighlight %}

And your `UITableView` is fully configured with that little amount of code in your view controller. This clearly helps with MassiveViewController syndrome, even if it merely hides the code else where. But it does allow for extremely efficient reusability and customization. Lets say you want to make another view controller to show information about the authors. This is the entirety of the code you have to add:

{% highlight swift %}
struct Author {
  var name: String
  var birthDate: Date
  var birthDateString: String { // dateformatter shenanigans }
}
class AuthorCell: UITableViewCell { // set up views }
extension AuthorCell: ConfigurableCell {
  var identifier: String { return "AuthorCell" }
  func configure(for object: Author, at indexPath: IndexPath) {
    textLabel?.text = object.name
    detailTextLabel?.text = "Born on: " + object.birthDateString
  }
}
class AuthorDataSource: ArrayDataSource<Author,AuthorCell> {
  override func setup() {
    super.setup()

    tableView.rowHeight = UITableViewAutomaticDimension
    tableView.estimatedRowHeight = 55
  }
  func tableView(_ tableView: UITableView, viewForHeaderInSection section: Int) -> UIView? {
    let headerView = UIView()
    // setup headerView
    return headerView
  }
  func tableView(_ tableView: UITableView, heightForHeaderInSection section: Int) -> CGFloat {
    return 45
  }
}
class AuthorTableViewController: UIViewController {
  let tableView = UITableView(frame: CGRect.zero, style: .plain)
  var dataSource: AuthorDataSource!

  override func viewDidLoad() {
    super.viewDidLoad()
    let authors = APIClass.magicMethodThatGetsAuthors()
    dataSource = AuthorDataSource(tableView: tableView, array: authors)
  }
}
{% endhighlight %}

And that's it! With various other subclasses/protocols added in, code reuse can be further expanded upon. A further step is to make the `UIViewController` itself generic. Though, I have to admit, maximizing code reuse can be an addiction that sometimes Swift's compiler can't keep up with. I've noticed a few times that extensively generic view controllers utilizing generic properties can causes it to fail without any error in the code. But hopefully that's all fixed by the time Swift 4 comes out, though I'm not getting my hopes too high.

I also want to note that my biggest reference for this post was Objc.io's Core Data book. It introduced me to some of these patterns. I've since moved onto using Realm for all my local storage purposes, but that book had some really fascinating Swift code. 
