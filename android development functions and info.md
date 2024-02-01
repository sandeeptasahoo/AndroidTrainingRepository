## Toast message and Snackbar message
Toast message:
Toast message execute like a notification and though application get close or not the toast msg will remain their irrespectively 
Syntax: 
Toast.makeText(<activity>,<msg string>,time).show();
Eg:
Toast.makeText(this, “this msg displayed”, Toast.LENGTH_LONG).SHOW()

### SnackBar:
This msg will execute in app interface only if app get closed the text bar also gets closed
Syntax:   Snackbar.make(<view variable that’s being passed by onclick function>,<string msg> ,time).show()
Eg:   Snackbar.make(view,"The button is working",Snackbar.LENGTH_LONG).show()
2.	Logs
Used to see the log statements in logcat screen in android studio
Header: android.util.Log
Multiple types of msg can be thrown based on type 
Syntax: Log.i(<tag string>,<msg>)
I stand for info except that error ,debug etc type of string exist
3.	Access all xml variable through id
In gradle: module: app file add below plugin:
1.	id 'com.android.application'
2.	id 'kotlin-android'
3.	id 'kotlin-android-extensions'
import library : import kotlinx.android.synthetic.main.activity_main.*
(to access different different xml files we can change the name of it)
now we can access any object by its id of the container

Normal access also can be possible:
findViewById(R.id.headline)

Download Image from URL
URL(<image url string>).openStream().use{
	BitmapFactory.decodeStream(it)
}

## Alarm Manager
link: Android Example : Alarm Manager Complete Working – Android Clarified (wordpress.com)
Alarm manager helps to schedule a task to run at a specific time in future though mobile is running or not. 

For accessing the alarm service initially alarm manager is to be created
val alarmManager:AlarmManager=getSystemService(ALARM_SERVICE) as AlarmManager
 here by accesing system service we can access any service

pending intent: intent is used to transfer data and opening an activity from one activity but when the application is closed and we want to open an activity then pending intent is used to open that application and that exact activity 

to make exact time broadcast we can use alarm manager set exact function but it will require a special user permission for sending broadcast at exact time which can be mentioned in the manifest file 

<uses-permission android:name="android.permission.SCHEDULE_EXACT_ALARM"
    />

Now the function can be called like this 
alarmManager.setExact(AlarmManager.RTC_WAKEUP,triggerTime,pendingIntent)


Broadcast receiver is an android component which receives notification from other broadcast sender for which it has registered itself 

For cancelling the pending intent we need to access the pendingintent first for that we can access it by request id on which we have created it and the flag should be exactly same as we have created it before. If it is a broadcast one then access the pending intent as getBroadcast rather get activity. 

val alarmManager: AlarmManager =getSystemService(ALARM_SERVICE) as AlarmManager
val intent2=Intent(this,NotificationReceiver::class.java)
val pendingIntent: PendingIntent = PendingIntent.getActivity(this,data.id,intent2, PendingIntent.FLAG_IMMUTABLE)
Log.d("notification_log","the pending available is $pendingIntent is cancelled ")
if(pendingIntent!=null)
{
    alarmManager.cancel(pendingIntent)
}


## Calendar
For time assignment operation calendar variable is used to access current time or set any time to be trigger at a particular time
It takes 24 hour format
Error handle:
1.	the value set to the calendar in 12 hr format is handled carefully as 12.30 pm when converts to 24hr it becomes 00.30 which is abnormal with respect to normal nomenclature 
2.	the Calendar.MONTH starts from 0 

Make an option button
Creating a new xml file by with type casting menu
 

Then there we can create multiple option as needed 
 
Now for giving the access of the option button to the activity and create it in the UI
override fun onCreateOptionsMenu(menu: Menu): Boolean {
    menuInflater.inflate(R.menu.option_of_task,menu)
    return super.onCreateOptionsMenu(menu)
}

Now for giving functionality to the buttons 
override fun onOptionsItemSelected(item: MenuItem): Boolean {
    return when (item?.itemId)
    {
        R.id.Task -> ShowThisTypeTask("Task")
        R.id.List -> ShowThisTypeTask("List")
        R.id.Share -> ShowThisTypeTask("Share")
        else -> super.onOptionsItemSelected(item)
    }

}

Share intent between application
private fun ShareAppLink() {
   val intent=Intent().apply{
       action=Intent.ACTION_SEND
       putExtra(Intent.EXTRA_TITLE,"The Task is To be Done")
       putExtra(Intent.EXTRA_TEXT,"Open TODO APP")
       type= "text/bold"
   }
    startActivity(intent)
}

when ever this function is called the application broadcast as send action and whichever application support send action will catch it and will be displayed in the selection menu and by extra bundling with proper naming we can send data to other app

Make a back button
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.input_task)
    supportActionBar?.setDisplayHomeAsUpEnabled(true)
 
supportActionBar?.setDisplayHomeAsUpEnabled(true) : this line of code able to display a preset back button on the top section of UI which enables to make the page go back to the parent activity but for that in manifest file we need to mention the parent activity of the current activity 
<activity
    android:name=".TaskInput"
    android:parentActivityName=".MainActivity"
    android:exported="false"/>
<activity
    android:name=".ViewTheTask"
    android:parentActivityName=".MainActivity"
    android:exported="false"/>
<activity
    android:name=".ThingsList"
    android:parentActivityName=".MainActivity"
    android:exported="false"/>

## UI components:
1.	Navigation draw menu:
a.	Add icon to each item:
b.	Header to navigation draw menu:
c.	Navigation toggle button:

AIDL ( Android interface definition language)
1.	Marshalling and Demarshalling (it is similar to serialising and serialising)
2.	STUB 
3.	onServiceConnected() this will return the binder

broadcast receiver 
to receive broadcast a class is needed which should be extent to super class BroadcastReceiver in which an abstract function is to be defined which is OnReceive
In manifest file we need to declare for which type of broadcast the receiver will react and which is the broadcast receiver . its also important to declare request for permission for certain broadcasts. 

This above type of setting of broadcast receiver is IMPLICIT type receiver where we are setting the receiver to receive or react to specific type of system broadcast.
Its also a static type broadcast receiver where we declare the receiver in the manifest file  and the specific receiver will listen to the broadcast while its in the background also.
IMPLICIT broadcast system has deprecated from android oreo and above series so EXPLICIT broadcast system is used.

The OnReceive function has two parameters 
1.	intent:
a.	this parameter show on which action intent the function is triggered. By using this we can react to specific action.
b.	Eg: if(Intent.action_Boot_Completed.equals(intent.getAction()))

Dynamic receiver are the receivers which receives broadcast while the application is running in the foreground. Here the receiver is not needed to be declared in the manifest files, instead its dynamically created in the main activity.
Here in life cycle we want to listen to the broadcast while the application is running so we need to register the receiver when the application starts and deregister when the application stops.
 

In EXPLICIT broadcast receivers we create a receiver which listens to the broadcast from another application which sends the broadcast mentioning specific action. This broadcast action name should be unique as it’s a global declaration.

 

In the broadcast sending section we need to declare the broadcast action name through an intent and then broadcast it through sendbroadcast function. 
 

In EXPLICIT receivers the broadcast sender can trigger specific receivers through class name,package name. Here the receiver should be register in the manifest so that other broadcaster should access it.
Ex:
1.	When broadcast and receiver in same app: 
2.	When broadcast and receiver are in different app:   here the receiver should be register in manifest file where a parameter exported is set true as other application can access it. By making it false we are adding a restriction that no other application can access it. 
3.	When broadcast and receiver are in different app and one broadcast is accessing multiple apps. The receiver should declare in all their manifest file with intent filter and when the broadcast is sent all the apps that are having this action will trigger.    

In ORDERED broadcast , the broadcast will access the receivers sequentially of an application. The order will be as first it will access the dynamic receiver then will access the receivers as mentioned in the manifest file.
  

The broadcast receivers order can be customised by giving priority value to each higher the priority the receiver will access first.
  

We can access the receivers sequentially by sendOrderedBroadcast function and we can send and alter data through this also.  
Here senderreceiver is the last receiver that will trigger and the data packets that is passed is initial code,initial data and extra bundles.
  

Here setresult function is used to set all the data to the intent.
We can stop the receiver cycle at any receiver point by abortBroadcast() function in the OnReceive body.

The broadcast receiver can restrict certain broadcast by asking for custom permission as we can set in manifest file a receiver can only trigger when the broadcaster has certain permission. As follows in manifest folder at receiver end.  

Same can be achieved by sender end as we can restrict receiver to receive the broadcast if it does not have some specific permission.

 

Threading in receivers are required when we are doing heavy work on receivers as after broadcast the receiver returns within 10 sec so that there is no heavy load in background but if we want intentional work more than 10 seconds pending intent is needed to set and tread.

 


Local broadcast manager is deprecated but it’s an easy concept that states communication within the app.


Services:
Service is android provided process to make application run in the background.
Its of two types:
1.	Strat service:
2.	Bound service:
3.	Foreground service:

## HaMer
This architecture states about an execution pattern where all computation is done in a different tread making main thread free for UI operation. This is very convenient and user friendly way of interaction.

For this we need to create a runnable which encapsulates the background thread work. And by using this runnable we can implement the thread anywhere. 
   

This runnable can be executed through any thread or any process.
For executing in the main thread we can run the runnable directly by command runnable.run() 

A thread is also created for execute the task but its not preferred as multiple threads in thread pull will slow down the process. 


The class declaration and thread runnable creation can be done simultaneously for reducing the code length  

We cant update the UI from any background thread we need to follow some method for update.
Now if we need to update the UI from this thread we need to use following method:
1.	Using handler:    handler is needed to be created in the mainthread and its used for posting and updating the UI. If we wanted to create the handler in the background thread but we need to post to the UI we need to mention the thread in the mainloop. 
2.	Using direct View component for posting:  
3.	Using direct runOnUithread :  


Mechanics of a thread and handler execution:
 

The thread has three part 
1.	Start: once the tread is start we can post task to the thread and once the tread is terminated we cant start the same thread again.
2.	Loop: looper is need to define for a thread individually , if a looper is not define to a thread it will terminate after doing the assigned task after the start command. If a looper is defined then task is needed to be posted to the looper queue so that the looper will do the task. The assigned task to the looper can be posted with delay making the task sequence customisable. 
3.	Terminal: when a looper is assigned to the thread its important to terminate the thread in the onStop lifecycle so that there wont be any memory leak and there is no pressure to the processor.

The handler is the operator that sets the message queue of task and performs every task that is assigned to the queue. So when a task is assigned to the looper it arranges its priority queue and when the task is needed to perform it feeds the task to the handler to perform the task. 

A handler assigns to the thread where its been instantiated.
A handler cats be assigned to a thread having no looper.
A handler is used to post task (runnable) to the thread.

   

Better execution of this type of structure is creation of msg and sending them to the thread.
     

Handlerthread:
It’s a thread which has a looper attatched to it so here no need to declare looper separately but need to create a handler within it in the prepare loop section 
1.	Creating handlerthread directly: its not a good practice as during rotation when the lifecycle destroys and restarts the previous activity is not able to destroy as the thread was accessing the data of that activity which creates memory leaks.     there are several function for posting the runnable in different method and with different time delays. But when we post any runnable without any delay it will instantiate immediately. So if we post something in the front of the queue after posting something without delay then the execution will be occur for the first post rather the task that has been sent to the front of queue.      

2.	Creating a threadhandler through a class:   here a priority can be given to a thread so that the system can know the task’s importance.  

Removing message or runnable from queue
  

null parameter removes all the msgs from the queue but if we want to remove specific parameter then we need to pass the instance of that in case of runnable and in case of msg we need to pass the value of ( What) parameter.   





## Coroutines
Advantages of coroutines over threads
1.	Coroutines uses same thread multiple times from the thread pool so they are “ not resource intensive “ that is they don’t take much memory and its “Light weight”
2.	It has multiple feature and synchronisation is easy. Asyn to syn conversion is easy.
3.	A coroutines can be pause and resume any time.
4.	Data syn is easy.
Scope:
Scope provides lifecycle methods for coroutines. It also allow us to start and stop coroutines
3 types of scope present:
1.	GlobalScope.launch:
a.	The scope remains the entirety of the application. It only get destroyed when the application get destroyed. 
2.	runBlocking:
a.	it creates the scope blocking the executing thread so mostly used to block a coroutines and start executing some code.
3.	coroutineScope: 
a.	mostly used and creates a scope for coroutines and continues till all children coroutines get completed.
  
Creating A coroutine that dies after sometime 
 
Context:
It contains the data regarding the coroutines scope as eg: name of thread and etc.
It has 2 important element.
1.	Dispatcher – tells about the thread the coroutines runs on 
2.	Job – handles the coroutine lifecycle
 

Suspending function:
Coroutine can execute suspended function only. And suspended function can only synchronise the variable.

 
Job
Jobs are lunched coroutines it gives ability to manipulate coroutine life cycle.
We can create hierarchy of jobs. When job is cancelled all children as well as parent jobs will be cancelled.
        
  
 
Dispatcher
It defines the thread with which the coroutine will going to work.
5 types of dispatcher:
1.	Main: used to update UI related word;
2.	Default: Used for CPU intensive work
3.	IO : Used for network communication , reading and writing work
4.	Unconfined : used the same thread the parent is using.
5.	newSingleThreadContext(“Mythread”) : creates a new thread ( usually not preferred )

   
Async
Its like lunching a coroutine where we expect a return value.
As An async coroutine might have different speed of execution we need to wait till we get the value which is the Deferred value.
For that we use await() function.

   
withContext
used to change the context with the dispatcher
 
Exception handling:
In normal lunched coroutines exception handling coroutine can be defined to throw exception while they occur while in Async type coroutine we need to use try and throw block to handle the exception as the deferred value is being waited for.
    

Gradle dependency
Add these dependencies to your module level (app) build.gradle file
1.	implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-core:1.3.0'
2.	implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:1.3.0'

Important Analysis :
1.	If main thread(in switch the scope is created) get destroyed then the coroutines also get destroyed.
2.	The created/started thread must required to be destroyed to avoid memory leak. Run() execution completes the task and destroys the thread in runnable. If run() has some network calls and the network call is not returned then the curser wont return and lead to memory leak.

Shared State
When one variable is used in multiple coroutines which can create problem in updating the value  
Here the counter should be 100000 but for some overlapping process the value reached 51223

1.	Atomic value: the value can be used for integers, Boolean but not for complex variable. But shared state works perfectly.   
2.	Finegrain method: here change to variable work on a single thread or context only but the process or function can run on a different thread. As context switching takes place very rapidly the process takes more time.  
3.	Coursegrain method : when the process of changing the variable works in a single thread. Here process takes less time but sometimes its not possible to run all process in a single thread.   
4.	Mutex method: here the changing variable process can be execute as lock and unlock process. When some coroutine is accessing the variable the mutex lock the variable so that no other coroutine can update it. The process gives best performance in execution time and correctness.  

## FLOWs
1.	Flow is a container which notifies the subscriber about any addition of element into it.

 

2.	A list can be converted to flow 

 

3.	Can be created directly as flowOf(1,2,3)
4.	 Until collect function is not called with flow variable there wont be any emit from flow.
Operations on flow: 
1.	Map : used to convert every data point to some format  
2.	Filter: used to remove some value from flow following some logic  
3.	Transform: it is used to do operation on data without affecting the original data  
4.	Take: limit operation to first N number of values  
Terminal operators: 
a.	Collect
b.	toList
c.	toSet
d.	reduce: reduce operator operates with value and store the result in accumulator  
5.	flowOn : used to change the thread
  
6.	buffer: when emiting and collecting has some time delay its needed to collect the new emited items while the last collected item is being processed.   
7.	zip: its used to combine two flows with some functionality but the zip function matches each corresponding index of two flows.  
8.	combine: it is used to combine two flows as the last emited value of one flow to last emited value of other  
Important operators:
Flow.first{Boolean condition}: if condition gives true for the first time it will emit that value only not others
catch:
onCompletion: 


## Channels:
1.	Channel is a queue of data
2.	A coroutine can asynchronously put elements.send(data)
3.	Another can blockingly get elements.receive()
4.	A channel is closed when there are no more elements expected .close()   
5.	CoroutineScope.produce{…}: creates a data source in a scope 
 
6.	PipeLine: When data received from one channel will be getting used in other.
 
7.	Fan out: if multiple coroutines are receiving data from same channel then the data will be receive by each scope in equal distribution to reduce load. This is only valid when data production rate is higher than the data access means when one scope is busy the data will be send to different coroutine  
 

8.	Fan in:
When from multiple coroutine data is passed to same channel it will act as queue as the way it receive the data it will emit data similarly.  
9.	Buffered channel: the channel can have a capacity and will send or store the data till that capacity after that it blocks sending data once data starts receiving it releases the data which enables to send more data  
10.	Ticker channels: it has two parameter initial delay and periodic delay. When channel starts it waits initially for sometime and then send data with periodic time lapse.  
