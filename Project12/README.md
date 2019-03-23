## Day 48
# Saving Data
It is finally time to learn how to save our data! Great! This was long overdue! 
iOS makes it a breeze to save data! Well… at least for the users, because some great developers spent their time finding a way to make them forget about having to do so!
I still remember the dreadful days as a Windows user (or when I use Office for Mac, even the 2019 version, or any other app for Mac not natively developed on Mac) when I was quitting a program and getting the notification that if I had not saved my progress things would have been lost! 
## Setting up
I’m trying something new myself here: in Xcode, I am branching from the master branch to a new branch called “project12a”. 
Hopefully all my changes will be saved in the new branch and when I will want to go back I will just have to switch to the “master” one. 
Completely new to this so… wish me luck! 
## Reading and writing basics: `UserDefaults`
This first part was simply about getting to know the `UserDefaults` and its workings. This is a subclass of `NSObject` described as “an interface to the user’s defaults database, where you store key-value pairs persistently across launches of your app”. This part from the Documentation’s Discussion seems important to me:
> At runtime, you use `UserDefaults` objects to read the defaults that your app uses from a user’s defaults database. `UserDefaults` caches the information to avoid having to open the user’s defaults database each time you need a default value. When you set a default value, it’s changed synchronously within your process, and asynchronously to persistent storage and other processes.
The `.standard` class variable returns the shared defaults object and, if it doesn’t yet exist, it creates one. We can then store all this in a property and use it to write some sets of preferences using the `.set(_ value: Any?, forKey defaultName: String)` method.
If we want to read preferences, instead, we need to use the `.objectType(forKey:)` method, filling objectType with anything we may be looking for, from `Int` to `String` to whatever else. 
Directly using the `.object(forKey:)` creates some troubles because we receive an optional value in return and we need to downcast it and, possibly to _nil cohalesce_ it to a default value in case it may not exist. 
## Fixing Project 10: `NSCoding`
Let’s make some more magic now: we learn that we can save just any kind of data into `UserDefaults`, as long as we follow some rules. In short:
- we use the `NSKeyedArchiver` class which is defined as _“A coder that stores an object's data to an archive referenced by keys”. _
	- we call its `archiveData` method which, strangely, is described as _deprecated_ in the documentation… but what is taking its place? Whatever, this transforms an _object graph_ (i.e.: that object plus all its references) into a `Data` object and writes it to `UserDefaults`. 
- data can be of any type as long as it is one of the simpler types or if it conforms to the `NSCoding` protocol. 
So, in project 10, we add conformance to the `NSCoding` protocol at the top of the `Person` class. We are explained that structs cannot conform to `NSCoding`… I am not sure I understand this, but maybe I don’t have to. I will just tell my brain that “if I want to save something I have to make that something a class that inherits from `NSObject` and that conforms to `NSCoding`”. Period! 
Protocol conformance brings with it its issues and so we need to add the `encode` method which will encode (i.e.: **save**) our name and image behind properly named keys (it looks like a dictionary and maybe it just is) and a `required init`—that is an initialiser that subclasses will need to add as well if they inherit from this class—with an assignment to our `name` and `image` properties to a call to the `decodeObject(forKey:)` method. We also decide to play it safe and add a _nil coalescing_ operation in case the data is not found. 
In short: 
> the initialiser is used when loading objects of this class and `encode()` is used when saving
Back in _ViewController.swift_ we now have to write the actual code that saves and loads but we need `Data` and we have arrays so… off… we need to convert things first. 
Luckily for us we “just” need to add a new `save()` method where we optionally try to store the return result of the call to `NSKeyedArchiver.archivedData` on our `people`’s array into an optionally bound constant and, if that succeeds, we will set that to be saved in our user defaults storage! Easy right? …yeah…!
Now we need to call `save()` wherever we were reloading our collection-view data and, finally, to allow for loading in `viewDidLoad()` so that when we launch the app the next time we will actually load the same data as the last time. 
So, in `viewDidLoad()`, we optionally bind the return object of the `defaults.object(forKey:)`—using the same key we used to save the data, of course—to a constant and optionally downcast it to a `Data` object. If this succeeds we optionally try to store the result of the call to `NSKeyedUnarchiver.unarchiveTopLevelObjectWithData()`—did you try to say it in one breath— optionally downcast as an array of `Person`s in a constant and, if also this succeeds we set our `people` variable to this last constant! If, God forbid, anything would go wrong, we just load an empty `[Person]`. 
## Fixing Project 10: `Codable`
If we are only writing Swift, `Codable` is the way to go. The differences between `Codable` and `NSCoding` are the following:
- `Codable` works on both classes and structs.
- No need to write `encode()` and `decodeObject()` ourselves.
- Native support for JSON read/write.
Not bad, right?
First, I checked-out to my `master` branch and created a new branch from it called `project12a` (starting to feel dangerous power flowing through my veins!) and added conformance to `Codable` for the `Person` class. And actually there’s nothing else here to do…! Amazing! 
Back in _ViewController.swift_ we create a `save()` method. Inside, we declare and initialise a new `JSONEncoder()`, optionally try to (optionally) bind the result of the call to that encoder’s `encode` method to a constant and, if that succeeds, we set the saved data for the appropriate key in our user-defaults standard storage. Just for safety, if something goes wrong, we print a message to the console.
As before, we call this `save()` method everywhere we reload our data. 
We then move to `viewDidLoad()`, create a call to our `UserDefaults.standard` path, optionally bind the optional downcast to `Data` result of the call of the `object(forKey:)` method on our “people” key and, if that succeeds—breath!…—we declare a new `JSONDecoder()` and set up a `do-catch` block to try to create an array of `Person` objects from the data extracted before. 
This very last passage deserves a little clarification: the `object(forKey:)` method pulls out optional `Data` which gets unwrapped with `if let` and `as?`. The `JSONDecoder` then converts it back to an object graph—that is, our `[Person]`.
----
That’s it! You can find the code for all the three versions of the project [here]. 
Please let me know if something is amiss or suspiciously wrong.
Please don’t forget to drop a hello and a thank you to _Paul_ for all his great work (you can find him on [www.twitter.com/twostraws]) and don’t forget to visit the _100 Days Of Swift_ initiative [www.hackingwithswift.com/100]. 
