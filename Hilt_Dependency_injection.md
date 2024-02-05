#### link of course : https://samsungu.csod.com/ui/lms-learning-details/app/onlineContent/9920475f-f319-5f2b-b12a-cd005a776d8e
#### link of android doc : https://developer.android.com/training/dependency-injection/hilt-android

## introduction: 
#### what is dependecy injection?
dependecy injection is a design partern so its independent of platform so each platform handles it differently.
Dagger2 is the dependency injection from android frame work.

## Dagger:
Module ( @Provider ): it will have the instantiation of the class to inject 
Component (fun Inject(to)): method are provided to inject the classes
Target (@inject): injection is done on the targeted class

## Hilt:
In hilt the component is not created by user the platform will create the component on its own
### hilt is supported with following classes
1. Application
2. Activity
3. Fragment (not supported on retained fragments)
4. view
5. Service
6. BroadCast Receiver
7. ViewModel (by using one extension)

### Dagger hilt dependecy addition to your project
at application gradle setting add dependency : classpath "com.google.dagger:hilt-android-gradle-plugin:${ libs.versions.dagger.get()}"
at module gradle setting add dependency and plugin: 
dependencies {
  implementation "com.google.dagger:hilt-android:2.44"
  kapt "com.google.dagger:hilt-compiler:2.44"
}
plugins {
  id 'kotlin-kapt'
  id 'com.google.dagger.hilt.android'
}
### Hilt architecture 
for hilt, there should be an entry point to the application.
An application class is required with Hilt annotation 

[make sure to declare it in android manifest] as
<application 
android:name=""
</application>

eg:  
@HiltAndroidApp
class MainApplication : Application(){
}

### Injection
#### Constructor injection
when we require a class to be injected in other class we inject the constructor for ToBeInjected Class, then only we can inject that class to main class

eg: class TobeInjected @inject constructor(val value:int){}
    class MainClass @inject constructor( val injectClass: TobeInjected){}
    
#### Field injection 
field injection can be done in classes or activities which are annoted with @AndroidEntryPoint 
and it can inject such classes which have Constructor injection 

eg:
@AndroidEntryPoint
class MainActivity : AndroidComapatClass{
@inject
var lateinit mainclass : MainClass
}

#### method injection
method injection will execute while injecting the method to target 
the method injection will help to access the classes which have constructor injection 
this will help in scenarion where class need to be accessed

eg: 
@inject 
fun methodInjection( mainClass: MainClass){
mainClass.doSomeThings()
}

### Module
Module in hilt is different with compare to android architecture structure.
Modules gives command to hilt how to create the interfaces / how to bind intefaces.

#### Bind
can be implemented in a abstract only and bind gives hilt a idea which instance the injector is calling.

eg: 
@Module
@InstallIn(ActivityComponent::class)
abstract class Network{
    @Binds
    abstract fun bindNetworkAdapterImpl(networkAdapterImpl: MyAppNetworkAdapter): NetworkAdapter

}

This will give access to all classes, which has inheritance from ActivityComponent, access to access NetworkAdapter injection 
