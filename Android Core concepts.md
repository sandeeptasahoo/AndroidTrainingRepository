## Content:
1.	Data binding
2.	Navigation
3.	Paging
4.	Work manager
5.	Lifecycles
6.	Live data
7.	Room
8.	View  model
## Android application life-cycle
 
Whenever the layout style changes as portrait to landscape or landscape to portrait the life cycle ends and starts again
ha
The life cycle can be handle by an external class with observer in main class which can be accessed by onlifecycleEvent for that a class is to be created with inheritance of LifecycleObserver and define by annotations

import android.util.Log
import androidx.lifecycle.Lifecycle
import androidx.lifecycle.LifecycleObserver
import androidx.lifecycle.OnLifecycleEvent

class lifecycle: LifecycleObserver {
    @OnLifecycleEvent(Lifecycle.Event.ON_CREATE)fun onCreateEvent()
    {
        Log.i("app lifecycle logs","app created")
    }

}

In oncreate function In main.kt place following function call
lifecycle.addObserver(lifecycle())

(This library file is old)

View model
This is the most important part of app architecture component where we can structure our database and can handle live data

It needs dependency in build.gradle(module:app) file
 implementation 'androidx.lifecycle:lifecycle-extensions:2.2.0'

basically view model gives us support on database management in a class format so we can create different data base or tables for different section of data in our app 
view model created when the application start as basically it’s a class and whenever it is created it will remains until it is not destroyed

the view model class:
package viewModel

import android.app.Application
import android.util.Log
import androidx.lifecycle.AndroidViewModel
import androidx.lifecycle.MutableLiveData

class NormalDataBase(app:Application) : AndroidViewModel(app) {
    var haveCreated=MutableLiveData<Int>()
    val context=app
    init
    {
        haveCreated.value=0;
        Log.d("ViewModel","view model created for $haveCreated times")
    }

    fun increament()
    {
        val temp:Int=haveCreated.value?:0
        haveCreated.value=temp+1
        Log.d("ViewModel","Have increamented")
    }
}

here as a variable is made we declare it as MutableLiveData which mean any change with the variable will reflect directly to the UI irrespectively of calling in the oncreate function and without assigning it to the variable

any mutablelivedata is nullable data so used accordingly with elvise operator
lateinit var normalDb:NormalDataBase
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)
    normalDb= ViewModelProviders.of(this).get(NormalDataBase::class.java)
    normalDb.haveCreated.observe(this, Observer{textView.text = "hello world $it" })
    Log.d("lifecycle","the app is created")
    numberOfDestuction= savedInstanceState?.getString("Number of destuction")?:"0"
    button.text="the layout destroyed $numberOfDestuction"
}

once a class is declared we have initiated the database as ‘normaldb’
now we need to pass an observer who will update the live data to system and it can be a lamda function

## Recycler View
List viewer is a list of views which contains several views and bind them in a list when the screen is scrolled the views in list do not destroyed it remains in the memory which increases execution time and makes the process slow. So the recycler view removes the view contain which are not on the screen making the function faster

The process continues with following flow

Activity->API call ->DATA ->Adapter -> Viewholder -> recyclerView<-Layout Manager

After fetching data through API the data is provided to adapter then from adapter and view holder the data is set to the view then provided to the recyclerView . the layout manager will maintain the recycle view for process   

Initially in main context a layoutmanager is declared as recycle viewer with type of required layout (option are grid, linearlayout…) which is a access to the recycler viewer where we will add data 
Rv.layoutManager=LinearLayoutManager(contex:this)

Now adapter is created as a class which will assign object to recyclerviewer and this will require a view holder which is basically a layout item which depicts what will the recycle viewer will hold for it another class will be created 

Now an xml file is created which contains the view design of each view holder and this will convert to item view and that variable is passed to the adapter. The adapter requires data as parameter which it will pass as contain to the view holders

The adapter has 3 inplemented function 
Oncreateviewholder : the function binds the xml file to view item and returns it 
onBindiViewHolder :this function binds the data to viewholder
getItemCount: this function returns the number of data in data parameter passed

here the viewholder need to be created to bind the xml file through a class which will give access to the components that the xml file contains. This is possible by “LayoutInflater” 
in onBindViewholder we have access to particular view holder so we can assign data to it by the class made for view holder

Adapter class
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.TextView
import androidx.constraintlayout.widget.ConstraintLayout
import androidx.recyclerview.widget.RecyclerView
import com.example.recyclerview.R

class TaskAdapter(private val data:ArrayList<Map<String,String>>,private val onClick:onClickofTask ): RecyclerView.Adapter<TaskViewHolder>() {
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): TaskViewHolder {
        val view=LayoutInflater.from(parent.context).inflate(R.layout.task,parent,false)
        val viewHolder=TaskViewHolder(view)
        view.setOnClickListener{
           //Log.d("click check","${data[viewHolder.absoluteAdapterPosition]["header"]} is clicked")
            onClick.showTaskDetails(data[viewHolder.absoluteAdapterPosition])
        }
        return viewHolder
    }

    override fun onBindViewHolder(holder: TaskViewHolder, position: Int) {
        val currentData=data[position]
        holder.header.text=currentData["header"]
        holder.description.text=currentData["description"]
        holder.startTime.text=currentData["StartTime"]
    }

    override fun getItemCount(): Int {
        return data.size
    }
}

interface onClickofTask
{
    fun showTaskDetails(data:Map<String,String>,)
}

class TaskViewHolder(itemView: View): RecyclerView.ViewHolder(itemView)
{
    val task:ConstraintLayout=itemView.findViewById(R.id.TaskContainer)
    val header:TextView=itemView.findViewById(R.id.Heading)
    val description:TextView=itemView.findViewById(R.id.Description)
    val startTime:TextView=itemView.findViewById(R.id.starttime)

}
Main Activity
package com.example.recyclerview

import TaskAdapter
import android.content.Intent
import android.os.Bundle
import android.util.Log
import androidx.appcompat.app.AppCompatActivity

import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView
import onClickofTask


class MainActivity : AppCompatActivity(), onClickofTask {
    lateinit var recyclerView: RecyclerView
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        recyclerView=findViewById(R.id.RecycleViewer)
        recyclerView.layoutManager=LinearLayoutManager(this,LinearLayoutManager.VERTICAL,false)
        val data=fetchData()
        val adapter =TaskAdapter(data,this)
        recyclerView.adapter=adapter
    }

    private fun fetchData():ArrayList<Map<String,String>>
    {
        val data=ArrayList<Map<String,String>>()
        for(i in 1..100)
        {
            val map=mapOf(
                "header" to "task  $i",
                "description" to "description ${i*1000}",
                "StartTime" to "${i%12} AM",
                "EndTime" to "${(i+2)%12} AM"
            )
            data.add(map)
        }
        return data
    }

    override fun showTaskDetails(data: Map<String, String>) {
        //Log.d("clickCkeck","${data["header"]} is clicked")
        val intent= Intent(this, ViewTheTask::class.java)

        for(i in data)
        {
            intent.putExtra(i.key,i.value)
        }
        startActivity(intent)
    }
}

room data base and view model
Android Room with a View - KotlAndroid Room with a View - Kotlinin : instruction

Plugins required in build gradle(module)
plugins {
    id 'com.android.application'
    id 'kotlin-android'
    id 'kotlin-kapt'
}

key points:
1.	while accessing data from database we cant access it in main tread we need back ground tread to access it. But while assigning value to Livedata we cant assigned the value in the background tread so basically LiveData<>.value=… wont work in background for that we can use ‘postValue’ function to assign value to LiveData in a background thread



## Flow
Documentation link:
1.	Kotlin flows on Android  |  Android Developers
Video link:
1.	(111) Kotlin Flow API - Introduction to Flow - YouTube


Solid architecture:
Dagger injection: 

