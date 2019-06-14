# Core Data 

## [Demo app](https://github.com/joinpursuit/Pursuit-Core-iOS-CoreData-Recipes)

## Vocabulary 

- NSPersistentContainer
- viewContext
- Schema
- Data Model (File) 
- Entity 
- Attribute 
- Data Model Inspector (Module, Codegen, Relationship) 
- Editor Style: Table, Graph
- NSFetchRequest 
- NSManagedObject 
- NSManagedObjectContext
- NSPredicate
- NSSortDescriptor 
- performBackgroundTask
- perform
- saveContext
- NSFetchResultsController
- NSFetchResultsControllerDelegate

## Readings 

1. [Core Data Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CoreData/index.html#//apple_ref/doc/uid/TP40001075-CH2-SW1)  
1. [Core Data Framework](https://developer.apple.com/documentation/coredata)
1. [NSPredicate Class](https://developer.apple.com/documentation/foundation/nspredicate)
1. [Predicate Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Predicates/AdditionalChapters/Introduction.html#//apple_ref/doc/uid/TP40001789)
1. [Stackoverflow - Why should I use Core Data?](https://stackoverflow.com/questions/1883879/why-should-i-use-core-data-for-my-iphone-app)  

## Other Persistence mechanisms we have used

1. UserDefaults (saving simple data sets, e.g zipcode, background color of app) 
1. Using FileManager and Documents Directory (PropertyListEncoder, PropertyListDecoder) 
1. Firebase (online and offline persistence) 

## 1. What is Core Data 

Core Data is an object-graph relational database. 

## 2. How do you add Core Data to your app 

Be sure to check the box next to **Use Core Data**  

<p align="">
  <img src="https://github.com/joinpursuit/Pursuit-Core-iOS/blob/master/units/unit05/lesson-12-core-data/Images/adding-core-data.png" width="731" height="527" />
</p> 


After creating your new Project with Core Data, the methods below will get generated by Xcode.
```swift 
  func applicationWillTerminate(_ application: UIApplication) {
    // Called when the application is about to terminate. Save data if appropriate. See also applicationDidEnterBackground:.
    // Saves changes in the application's managed object context before the application terminates.
    self.saveContext()
  }

  // MARK: - Core Data stack

  lazy var persistentContainer: NSPersistentContainer = {
      /*
       The persistent container for the application. This implementation
       creates and returns a container, having loaded the store for the
       application to it. This property is optional since there are legitimate
       error conditions that could cause the creation of the store to fail.
      */
      let container = NSPersistentContainer(name: "CoreData_Recipes")
      container.loadPersistentStores(completionHandler: { (storeDescription, error) in
          if let error = error as NSError? {
              // Replace this implementation with code to handle the error appropriately.
              // fatalError() causes the application to generate a crash log and terminate. You should not use this function in a shipping application, although it may be useful during development.
               
              /*
               Typical reasons for an error here include:
               * The parent directory does not exist, cannot be created, or disallows writing.
               * The persistent store is not accessible, due to permissions or data protection when the device is locked.
               * The device is out of space.
               * The store could not be migrated to the current model version.
               Check the error message to determine what the actual problem was.
               */
              fatalError("Unresolved error \(error), \(error.userInfo)")
          }
      })
      return container
  }()

  // MARK: - Core Data Saving support

  func saveContext () {
      let context = persistentContainer.viewContext
      if context.hasChanges {
          do {
              try context.save()
          } catch {
              // Replace this implementation with code to handle the error appropriately.
              // fatalError() causes the application to generate a crash log and terminate. You should not use this function in a shipping application, although it may be useful during development.
              let nserror = error as NSError
              fatalError("Unresolved error \(nserror), \(nserror.userInfo)")
          }
      }
  }
```

The **name** below needs to match the name of the **.xcdatamodeld (Data Model)**   
```swift 
let container = NSPersistentContainer(name: "CoreData_Recipes")
```

## 3. The Data Model 

A look at the the Graphical Data Model editor. 

<p align="">
  <img src="https://github.com/joinpursuit/Pursuit-Core-iOS/blob/master/units/unit05/lesson-12-core-data/Images/graph-editor.png" width="671" height="447" />
</p> 

We control-drag from one entity to another to create a relationship. 

## 4. Entity and Attribute 

Analagous to Swift objects we have written an Entity is would be the class or struct and attributes would be the properties of that object. 

**Recipe Entity**    
Attributes: 
- calories 
- healthLabels
- imageURL 
- ingredientLines 
- label 
- yield 
- **relationship** source (to-one relationship)   

**Source Entity**  
Attributes: 
- name 
- **relationship** recipes (to-many relationship) 

## 5. Core Data Types 

Types: 
- Int16 
- Int32 
- Int64
- Double 
- String 
- Transformable 
- Decimal
- Float
- Date 
- Boolean 
- Binary Data 
- UUID 
- URI

## 6. Subclassing NSManagedObject 

NSManagedObject represents a record (object) in the database. We will create a subclass for each **Entity** in our Data Model. This gives us more flexibility into extending the objects inner functionality. Make sure to select **Category/Extension** in the Class section of the Data Model Inspector. 

Recipe class to represent the Recipe Entity

```swift 
import UIKit 
import CoreData 

class Recipe: NSManagedObject {
  // code here
}
```

Source class to represent the Source Entity

```swift 
import UIKit 
import CoreData 

class Source: NSManagedObject {
  // code here
}
```

## 7. Writing to Core Data 

1. Get the NSManagedObjectContext 
2. Use the Entity initializer 
3. Populate the newly initialized entity with appropriate attributes 
4. Create any relationships 
5. Lastly don't forget to save the context (NSManangedObjectContext) 

```swift 
let recipe = Recipe(context: context)
recipe.label = recipeInfo.label
recipe.healthLabels = recipeInfo.healthLabels.joined(separator: ",")
recipe.calories = recipeInfo.calories
recipe.imageURL = recipeInfo.image
recipe.ingredientLines = recipeInfo.ingredientLines.joined(separator: ",")
recipe.yield = Int32(recipeInfo.yield) // casting to Int32 for Core data use
```

Creating the relationship between the Recipe and Source (one-to-one relationship)
```swift 
// fetch the source
// create the relationship
do {
  recipe.source = try Source.createSource(recipeInfo: recipeInfo, in: context)
} catch {
  throw error
}
```

**Make sure to save the context**   
```swift 
try context.save()
```

## 8. Reading from Core Data 

1. Get the NSManangedObjectContext 
2. Create an NSFetchRequest 
3. Provide an NSPredicate and NSSortDescriptor as needed 
4. Execute the fetch request on the given context 

```swift 
if let context = container?.viewContext {
  let request: NSFetchRequest<Recipe> = Recipe.fetchRequest() 
  // predicate 
  // sort descriptor 
  
  // fetch base of given predicate and descriptor if provided 
  do {
    let recipes = try context.fetch(request)
    // here we should have an array of recipes if they have been saved prior
  } catch {
    print("fetching error: \(error)")
  }
}
```

## 9. NSFetchResultsController and NSFetchControllerDelegate

A controller (not in the view controller sense) that you use to manage the results of a Core Data fetch request and display data to the user.

**Configuring the NSFetchController**   
```swift 
private func configureFetchResultsController() {
  let request: NSFetchRequest<Recipe> = Recipe.fetchRequest()
  // predicate
  // use some example predicates
  //request.predicate = NSPredicate(format: "label contains [c] %@", "Salmon")

  // sort descriptor
  // use some example sort descriptors
  request.sortDescriptors = [NSSortDescriptor(key: "label", ascending: true, selector: #selector(NSString.localizedStandardCompare(_:)))]
  if let context = container?.viewContext {
    context.perform {
      self.fetchResultsController = NSFetchedResultsController<Recipe>(fetchRequest: request,
                                                                  managedObjectContext: context,
                                                                  sectionNameKeyPath: nil,
                                                                  cacheName: nil)
      do {
        try self.fetchResultsController?.performFetch()
      } catch {
        print("fetchResultsController performFetch error: \(error)")
      }
      self.fetchResultsController?.delegate = self
      self.tableView.reloadData()
    }
  }
}
```

**Using the fetchResultsController along with table view datasource methods to populate the table view**   
```swift 
extension SavedRecipesController: UITableViewDataSource {
  func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
    if let sections = fetchResultsController?.sections,
      sections.count > 0 {
      return sections[section].numberOfObjects
    } else {
      return 0
    }
  }

  func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    let cell = tableView.dequeueReusableCell(withIdentifier: "RecipeCell", for: indexPath)
    if let recipe = fetchResultsController?.object(at: indexPath) {
      cell.textLabel?.text = recipe.label
      cell.detailTextLabel?.text = recipe.source?.name
    }
    return cell
  }
}
```

**NSFetchResultsControllerDelegate Methods**    
```swift 
extension SavedRecipesController: NSFetchedResultsControllerDelegate {
  func controllerWillChangeContent(_ controller: NSFetchedResultsController<NSFetchRequestResult>) {
    tableView.beginUpdates()
  }
  
  func controller(_ controller: NSFetchedResultsController<NSFetchRequestResult>, didChange sectionInfo: NSFetchedResultsSectionInfo, atSectionIndex sectionIndex: Int, for type: NSFetchedResultsChangeType) {
    switch type {
    case .insert:
      tableView.insertSections([sectionIndex], with: .fade)
    case .delete:
      tableView.deleteSections([sectionIndex], with: .fade)
    default:
      break
    }
  }
  
  func controller(_ controller: NSFetchedResultsController<NSFetchRequestResult>, didChange anObject: Any, at indexPath: IndexPath?, for type: NSFetchedResultsChangeType, newIndexPath: IndexPath?) {
    switch type {
    case .insert:
      tableView.insertRows(at: [newIndexPath!], with: .fade)
    case .delete:
      tableView.deleteRows(at: [indexPath!], with: .fade)
    case .update:
      tableView.reloadRows(at: [indexPath!], with: .fade)
    case .move:
      tableView.deleteRows(at: [indexPath!], with: .fade)
      tableView.insertRows(at: [newIndexPath!], with: .fade)
    }
  }
  
  func controllerDidChangeContent(_ controller: NSFetchedResultsController<NSFetchRequestResult>) {
    tableView.endUpdates()
  }
}
```

## 10. How to add Core Data to an existing app

1. Copy the following into your AppDelegate 
<details>
  
  <summary>Core Data Stack</summary>
  
```swift 
  func applicationWillTerminate(_ application: UIApplication) {
    // Called when the application is about to terminate. Save data if appropriate. See also applicationDidEnterBackground:.
    // Saves changes in the application's managed object context before the application terminates.
    self.saveContext()
  }

  // MARK: - Core Data stack

  lazy var persistentContainer: NSPersistentContainer = {
      /*
       The persistent container for the application. This implementation
       creates and returns a container, having loaded the store for the
       application to it. This property is optional since there are legitimate
       error conditions that could cause the creation of the store to fail.
      */
      let container = NSPersistentContainer(name: "Name of Your Data Model Goes Here")
      container.loadPersistentStores(completionHandler: { (storeDescription, error) in
          if let error = error as NSError? {
              // Replace this implementation with code to handle the error appropriately.
              // fatalError() causes the application to generate a crash log and terminate. You should not use this function in a shipping application, although it may be useful during development.
               
              /*
               Typical reasons for an error here include:
               * The parent directory does not exist, cannot be created, or disallows writing.
               * The persistent store is not accessible, due to permissions or data protection when the device is locked.
               * The device is out of space.
               * The store could not be migrated to the current model version.
               Check the error message to determine what the actual problem was.
               */
              fatalError("Unresolved error \(error), \(error.userInfo)")
          }
      })
      return container
  }()

  // MARK: - Core Data Saving support

  func saveContext () {
      let context = persistentContainer.viewContext
      if context.hasChanges {
          do {
              try context.save()
          } catch {
              // Replace this implementation with code to handle the error appropriately.
              // fatalError() causes the application to generate a crash log and terminate. You should not use this function in a shipping application, although it may be useful during development.
              let nserror = error as NSError
              fatalError("Unresolved error \(nserror), \(nserror.userInfo)")
          }
      }
  }
```
  
</details>

</br>

2. Create a new Data Model file. File -> New -> File (Select **Data Model** below the Core Data heading)   
3. Make sure that the NSPersistentContainer name matches the newly created Data Model file. e.g if your Data Model file is called Company.xcdatamodeld then the name should be "Company". See below.

```swift 
let container = NSPersistentContainer(name: "Name of Your Data Model Goes Here")
```

