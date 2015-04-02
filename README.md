![AlecrimCoreData][image-1]

[![Carthage compatible][image-2]][1]

AlecrimCoreData is a framework to easily access CoreData objects in Swift.

## Minimum Requirements

- Xcode 6.3
- iOS 8.1.3 / OS X 10.10.2

## Installation

You can add AlecrimCoreData as a git submodule, drag the `AlecrimCoreData.xcodeproj` file into your Xcode project and add the framework product as an embedded binary in your application target.

## Getting Started

### Data Context

You can create a inherited class from `AlecrimCoreData.Context` and declare a property or method for each entity in your data context like the example below:

```swift
import AlecrimCoreData

let dataContext = DataContext()!

class DataContext: Context {
	var people:      Table<PersonEntity>     { return Table<PersonEntity>(context: self) }
	var departments: Table<DepartmentEntity> { return Table<DepartmentEntity>(context: self) }
}
```

It's important that properties (or methods) always return a _new_ instance of a `AlecrimCoreData.Table` class.

### Entities

It's assumed that all entity classes was already created and added to the project.

In the above section example, there are two entities: `Person` and `Department` (with `Entity` suffix added to their class names). You can name the entity classes as you like, of course.

## Usage

### Fetching

#### Basic Fetching

Say you have an Entity called Person, related to a Department (as seen in various Apple CoreData documentation [and MagicalRecord documentation too]). To get all of the Person entities as an array, use the following methods:

```swift
for person in dataContext.people {
	println(person.firstName)
}
```

You can also skip some results:

```swift
let people = dataContext.people.skip(3)
```

Or take only some results:

```swift
let people = dataContext.people.skip(3).take(7)
```

Or, to return the results sorted by a property:

```swift
let peopleSorted = dataContext.people.orderBy({ $0.lastName })
```

Or, to return the results sorted by multiple properties:

```swift
let peopleSorted = dataContext.people.orderBy({ $0.lastName }).thenBy({ $0.firstName })

// OR

let peopleSorted = dataContext.people.sortBy("lastName,firstName")
```

Or, to return the results sorted by multiple properties with different attributes:

```swift
let peopleSorted = dataContext.people.orderByDescending({ $0.lastName }).thenByAscending({ $0.firstName })

// OR

let peopleSorted = dataContext.people.sortBy("lastName:0,firstName:1")

// OR

let peopleSorted = dataContext.people.sortBy("lastName:0:[cd],firstName:1:[cd]")
```

If you have a unique way of retrieving a single object from your data store (such as via an identifier), you can use the following code:

```swift
if let person = dataContext.people.first({ $0.identifier == 123 }) {
	println(person.name)
}
```

#### Advanced Fetching

If you want to be more specific with your search, you can use filter predicates:

```swift
let itemsPerPage = 10  

for pageNumber in 0..<5 {
	println("Page: \(pageNumber)")
	
	let peopleInCurrentPage = dataContext.people
	    .filter({ $0.department << [dept1, dept2] })
	    .orderBy({ $0.firstName })
	    .thenBy({ $0.lastName })
	    .skip(pageNumber * itemsPerPage)
	    .take(itemsPerPage)
	
	for person in peopleInCurrentPage {
	    println("\(person.firstName) \(person.lastName) - \(person.department.name)")
	}
}
```

#### Asynchronous Fetching

You can also fetch entities asynchronously and get the results later on main thread:

```swift
let progress = dataContext.people.fetchAsync { fetchedEntities, error in
    if let entities = fetchedEntities {
        // ...
    }
}
```

#### Returning an Array

The data is actually fetched from Persistent Store only when `toArray()` is explicitly or implicitly called. So you can combine and chain other methods before this.

```swift
let peopleArray = dataContext.people.toArray()

// OR

let peopleArray = dataContext.people.sortBy("firstName,lastName").toArray()

// OR

let theSmiths = dataContext.people.filter({ $0.lastName == "Smith" }).orderBy({ $0.firstName })
let count = theSmiths.count()
let array = theSmiths.toArray()

// OR

for person in dataContext.people.sortBy("firstName,lastName") {
	// .toArray() is called implicitly when enumerating
}
```

#### Converting to other class types

Call the `to...` method in the end of chain.

```swift
let fetchRequest = dataContext.people.toFetchRequest()
let arrayController = dataContext.people.toArrayController() // OS X only
let fetchedResultsController = dataContext.people.toFetchedResultsController() // iOS only
```

#### Find the number of entities

You can also perform a count of the entities in your Persistent Store:

```swift
let count = dataContext.people.filter({ $0.lastName == "Smith" }).count()
```

### Creating new Entities

When you need to create a new instance of an Entity, use:

```swift
let person = dataContext.people.createEntity()
```

You can also create or get first existing entity matching the criteria. If the entity does not exist, a new one is created and the specified attribute is assigned from the searched value automatically.

```swift
let person = dataContext.people.firstOrCreated({ $ 0.identifier == 123 })
```

### Deleting Entities

To delete a single entity:

```swift
if let person = dataContext.people.first({ $0.identifier == 123 }) {
	dataContext.people.deleteEntity(person)
}
```

## Saving

You can save the data context in the end, after all changes were made.

```swift
let person = dataContext.people.firstOrCreated({ $0.identifier == 9 })
person.firstName = "Christopher"
person.lastName = "Eccleston"
person.additionalInfo = "The best Doctor ever!"

// get success and error
let (success, error) = dataContext.save()

if success {
	// ...
}
else {
	println(error)
}
```

### Threading

You can fetch and save entities in background calling a global function that creates a new data context instance for this:

```swift
// assuming that this department is saved and exists...
let department = dataContext.departments.first({ $0.identifier == 100 })!

// the closure below will run in a background context queue
performInBackground(dataContext) { backgroundDataContext in
	if let person = backgroundDataContext.people.first({ $0.identifier == 321 }) {
	    // must bring to backgroundDataContext
	    person.department = department.inContext(backgroundDataContext)! 
	    person.otherData = "Other Data"
	}
	
	backgroundDataContext.save()
}
```

## Using attributes and closure parameters

Implementation, docs and tests are in progress at this moment. A code generator utility is in internal beta and will be available soon.

## Branches and contribution

- master - The production branch. Clone or fork this repository for the latest copy.
- develop - The active development branch. [Pull requests][2] should be directed to this branch.

If you want to contribute, please feel free to fork the repository and send pull requests with your fixes, suggestions and additions. :-)

## Inspired and based on

- [MagicalRecord][3]
- [QueryKit][4]

---

## Contact

- [Vanderlei Martinelli][5]

## License

AlecrimCoreData is released under an MIT license. See LICENSE for more information.

[1]:	https://github.com/Carthage/Carthage
[2]:	https://help.github.com/articles/creating-a-pull-request
[3]:	https://github.com/magicalpanda/MagicalRecord
[4]:	https://github.com/QueryKit/QueryKit
[5]:	https://github.com/vmartinelli

[image-1]:	AlecrimCoreData.png?raw=true
[image-2]:	https://img.shields.io/badge/Carthage-compatible-4BC51D.svg?style=flat