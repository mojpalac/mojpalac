# Architecture

Interface and Classes that implements it: It's better to have FoodFactoryImplementation, then calling interface: FoodFactoryInterface(IFoodFactory etc.)

When constructors are overloaded, use static factory methods with names that describe the arguments

Don't use unnecessary information, if object is inside some module (SM), it does not require to call object SMFishCatalog

Functions should hardly ever be 20 lines long

We should avoid switch as they violate the Single Responsibility Principle and Open Closed Principle

Don't pass boolean as argument to function

if functions has more then 2 arguments probably it should have its own class


real goal of programmer is to tell the story of the system
Class structure should look like well written article From Headline to details

Class should expose "essence" of the data without showing implementation -> use interfaces for communications
Objects hide their data behind abstractions and expose functions that operate on that data
Data classes shouldn't have any meaningful functions
Data structure expose their data and have no meaningful functions.


//////////////// banners like that should be used hardly ever, otherwise it will be just noise

The Law of Demeter -> function talk only to friends

empty objects over null


|                                     Do                                     	|                                 Don't                                	|
|:--------------------------------------------------------------------------:	|:--------------------------------------------------------------------:	|
| ``` java Complex   fulcrumPoint  =  Complex . FromRealNumber ( 23.0 ); ``` 	| ``` java      Complex   fulcrumPoint  =  new   Complex ( 23.0 ); ``` 	|


Flow should be used for communication between Repo and ViewModel