# ANDROID PROGRAMMING UDACITY COURSE
# What is a content
Putting it simply:  
As the name suggests, it's the context of current state of the application/object. It lets newly-created objects understand what has been going on. Typically you call it to get information regarding another part of your program (activity and package/application).  
You can get the context by invoking ```getApplicationContext()```, ```getContext()```, ```getBaseContext()``` or this (when in a class that extends from Context, such as the Application, Activity, Service and IntentService classes).
Typical uses of context:  
* __Creating new objects__: Creating new views, adapters, listeners:  
	```java
	TextView tv = new TextView(getContext());
	ListAdapter adapter = new SimpleCursorAdapter(getApplicationContext(), ...);	
	```
* __Accessing standard common resources__: Services like LAYOUT_INFLATER_SERVICE, SharedPreferences:  
	```java
	context.getSystemService(LAYOUT_INFLATER_SERVICE);
	getApplicationContext().getSharedPreferences(*name*, *mode*);	
	```
* __Accessing components implicitly__: Regarding content providers, broadcasts, intent:  
	```java
	getApplicationContext().getContentResolver().query(uri, ...);
	```
# Part 2
### 2.1: Create a layout
* Create a LinearLayout and add object.
* Get the object with the ```findViewbyId(R.id.name_defined_xml)```
##### Notes on Data Binding
Instead of using findViewbyId, there is a more straightforward way:  
* First enable databinding in the gradle file: ```dataBinding.enabled=True```.
* Surround the xml file of an Activity with the <layout/> tag as the root tag.
* Then to get every element in the xml, just create a class "NameXmlFile + Binding", and get it with the command:
```java
XmlFileNameBinding mBinding = DataBindingUtil.setContentView(this, R.layout.xml_file_name);
```
* Then you can bind any value in the xml because they are now private argument of the class created
### 2.2: Add a Menu (a button in the ActionBar)
* Create a menu folder (in the res folder) and in it, an xml file which will be used to represend a Menu item.
* Override ```onCreateOptionsMenu``` in the main Activity file. It will modify the ActionBar by adding the new button just created.
```java
public boolean onCreateOptionsMenu(Menu menu) {
 MenuInflater inflater = getMenuInflater();
 inflater.inflate(R.menu.forecast, menu);
 return True;
}
```
* Set an action to execute when the Menu button is clicked:
```java
public boolean onOptionsItemSelected(MenuItem item){
	int id = item.getItemId();
	// Create a new intent to open up the Settings Activity
	if(id = R.id.action_refresh){}
}
```
* (Optionnal): A Toast message can be display with the command line:
```java
Toast.makeText(NameOfTheClass.context, message, TOAST.LENGTH_LONG).show()
```

### EditText
* Get the text of EditText widget: ```widget.getText().toString()```

### Connect to the Internet
* Don't forget to add permission in the Manifest. 

### AsyncTask
* Create a class which extended from AsynTask<URL, Void, String>. The three parameters are the type of the params send to the task, the type of the progress units published during the background computation, and the type of the result.
* Override the ```doInBackground``` function
* Override, (not forced), the ```onPostExecute```, which is the function called at the end on the computation. 
* To launch the async task, used the method ```execute(params)``` instead of ```doInBackground(params)```.

### addPolish
* Make a variable invisible in the UI using xml code: ```android:visibility```, then in .java file it is possible to modify the visibility by calling ```setVisibility(View.INVISIBLE)``` on the Object.
* Create a ProgressBar...

# Part 3. Recycler View
RecyclerView is responsible to display a List of elements which are display and scrollable. It optimizes the recycling of element out of scope. Here are the main steps for creating a RecyclerView:
* 1. Add dependencies in build.gradle ```compile 'com.android.support:recyclerview-v7:25.0.1'```
* 2. Transform ```ScrollView``` by ```RecycleListView``` in the layout.xml file (ScrollView is the old way to display a list).
* 3. Add a new layout in the layout folder which will represent an item in the list.
* 4. Create an ```AdapterViewHolder``` extending ```RecyclerView.ViewHolder```. It will cache children's views for an item. 
* 5. Create an ```Adapter``` class extending a ```RecyclerView.Adapter<AdapterViewHolder>```.
* 6. Override three functions:
	* ```onCreateViewHolder```: This gets called when each new ViewHolder is created:
    ```java
    public AdapterViewHolder onCreateViewHolder(ViewGroup viewGroup, int viewType) {
        Context context = viewGroup.getContext();
        int layoutIdForListItem = R.layout.recycler_view_id_in_xml;
        LayoutInflater inflater = LayoutInflater.from(context);
        boolean shouldAttachToParentImmediately = false;

        View view = inflater.inflate(layoutIdForListItem, viewGroup, shouldAttachToParentImmediately);
        return new AdapterViewHolder(view);
    }
    ```
	* ```OnBindViewHolder```: ```OnBindViewHolder``` is called by the ```RecyclerView``` to display one element of the List.
    ```java
    public void onBindViewHolder(AdapterViewHolder adapterViewHolder, int position) {
        String contentToDisplay = privateVariableData[position];
        adapterViewHolder.TextViewToModify.setText(contentToDisplay);
    }
    ```

	* ```getItemCount```: This method simply returns the number of items to display (The items data could be a ```String[]```, saved as a private variable in the Adapter class).
* 7. Modify data to display by creating a set in the Adaptater class, and call ```notifyDatSetChanged()``` in it.

##### Notes
* __Tag object__: The recyclerView offers something called a tag object, it's meant to store any data that doesn't need to be displayed. When calling ```onCreateViewHolder```, the tag can be set to any type of value like that: ```adapterViewHolder.itemView.setTag(objectToNotDisplay);```.

* __Swiping__: It is also possible to react when an element in the list is swipe right or left. Here is a simple code inside the onCreate of an Activity:
```java
new ItemTouchHelper(new ItemTouchHelper.SimpleCallback(0, ItemTouchHelper.LEFT | ItemTouchHelper.RIGHT) {
	// override when the item is moved	
	public boolean onMove(...){}
	// override when the item is swiped
	public void onSwiped(...){}
}
```

# Intents
### Definition
* Start an Activity from another Activity (in another app or not). It is done by passing messages called Intents. Note that further in the tutorial, Intent are also used to start Services, ContentProvider.
### Create an Intent
* To pass to another Activity, you have to create a new ```Intent```:
```java
Intent intent = new Intent(contextFirstActivity, contextNewActivity)
// A context object is a reference to the activity class (this or .class)
startActivity(intent);
```
### Pass and receive data
* To pass data, the method ```intent.putExtra("CODE_FOR_RETRIEVAL", dataVariable)``` can be used. The code is actually is used to retrieve the data later, as seen just next.
* To receive the data in the new Activity:
```java
Bundle extras = getIntent().getExtras();
data = (Type) extras.get("CODE_FOR_RETRIEVAL");
```
### Activity for Result
You can launch new activity only to get back result
```java
startActivityForResult(
	new Intent(),
	"CODE_FOR_RETRIEVAL");
// and get back the data
onActivityResult("CODE_FOR_RETRIEVAL", intResultCode, intent);
```

### Common intent 
* A common intent does not specify the app component to start, but instead specifies an action. Here is the example of how to start a browser. To check if an any app can resolve the activity, for example launch a browser, you can call the method ```intent.resolveActivity(getPacketManager())```. 
* When creating a new Activity it is possible to order them in the AndroidManifest.xml, so that the default Return button go back to the previous Activity. Here is the definition of a Child Activity:
```xml
<activity
            android:name=".DetailActivity"
            android:parentActivityName=".MainActivity">
            <meta-data
                android:name="android.support.PARENT_ACTIVITY"
                android:value=".MainActivity" />
</activity>
```

, and its parent Activity:
```xml
<activity
	android:name=".MainActivity"
        android:label="@string/app_name"
        android:launchMode="singleTop">
        <intent-filter>
           <action android:name="android.intent.action.MAIN"/>
           <category android:name="android.intent.category.LAUNCHER"/>
        </intent-filter>
</activity>
```
##### Notes
* ```launchMode``` tells whether the MainActivity should be recreated when getting back to the MainActivity or only restore.

# Life cycle
### Logging message to the terminal
* To log something, you can use the method: 
```java
// TAG can be Activity.class.getSimpleName()
Log.d(TAG, "string to print");
```
Other method exists depending the importance of the log (see the documentation).

### Lifecycle callback
![Activity Lifecyle](https://developer.android.com/guide/components/images/activity_lifecycle.png)

When the app started, stoped, or is even put in background, some functions fire and it's easy to override them:
* onCreate() : always implement this callback. It is called when the system create this Activity
* onStart()

### Save Activity state
Activity state can be destroyed for multiple reasons (pressing back button, calling finish()). If an Activity is destroyed and recreated, all the layout/view are saved using the Bundle instante (Bundle is a mapping from String keys to various Parcelable values, Parcelable is an interface to write and read back class). But __members variable of the Activity are destroyed__ so there must be a way to saved them:
```java
public void onSaveInstanceState(Bundle savedInstanceState) {
	super.onSaveInstanceState(savedInstanceState);
	savedInstanceState.putInt("CODE_FOR_RETRIEVAL", variableToSave);
}
```

Then, each variable can be restore in the ```onCreate()``` method using the Bundle.

### Use AsyncTaskLoader instead of AsyncTask
It makes ```AsyncTask``` non dependant of the Activity life. Because it becomes a Loader, __it will lives even if the Activity is destroyed__. For example if you launch a network research, and the orientation of the screen is changed, then the thread handler is reset but the thread keep running.
One particular subclass of Loaders is interesting: the ```AsyncTaskLoader```. This class needs us to override the same function as the AsyncTask, but is implemented a little bit better. It can handle Activity configuration changes more easily, and it behaves within the life cycles of Fragments and Activities. The nice thing is that the ```AsyncTaskLoader``` can be used in any situation where the AsyncTask is being used.

# Preferences
### Data Persistence
__How and what data should be saved, where?__
* Bundle to save key-value pairs. Data is gone if app is shutdown, it is a temporary saved place
* SharedPreference: save key-value pairs in a File. Save forever until the app is uninstall or phone is crashed.	It is used to save small information about the user/app state such as string/numerical values.
* SQLite
* Internal/External storage: save into the memory card/hard drive music...
* Save in the cloud (Google Firebase)

### Create a Settings Activity and add it a PreferenceFragment
* A ```PreferenceFragment``` is an object to display properly a settings parameter.
* Create a Menu item (saw before).
* Create a new class in the java folder called SettingsFragment inherited from ```PreferenceFragmentCompat```.
* Create an _xml_ folder, and an xml file in it. It creates by default a PreferenceScreen in the xml. You can add ```CheckBoxPreference```, ```ListPreference```... Here is an example:
```xml
<PreferenceScreen xmlns:android="http://schemas.android.com/apk/res/android">
    <CheckBoxPreference
        android:defaultValue="true"
        android:key="Name of the parameter"
        android:summaryOff="What to display when value is False"
        android:summaryOn="What to .. when value is True"
        android:title="Name to display"
        ></CheckBoxPreference>
</PreferenceScreen>
```
* Back in the SettingsFragment file, use the method ```addPreferencesFromRessource(R.xml.id_xml_file);
* Modify the settings activity xml file, if not done, to display the new PreferenceScreen.
```xml
<fragment xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/activity_settings"
    android:name="android.example.com.visualizerpreferences.AudioVisuals.SettingsFragment"
    android:layout_width="match_parent"
    android:layout_height="match_parent"/>
```
* Because we used the PreferenceFragmentCompat library, we need to add to the style xml:
```<item name="preferenceTheme">@style/PreferenceThemeOverlay</item>```.
### Access the parameter value from the main activity
#### Easy way
In a method called by ```onCreate()```, add the following code:
```java
SharedPreferences sharedPreference = PreferenceManager.getDefaultSharedPreferences(this);
// to get an object use the method get + type of the object
// Note that getString is used to access a string key from the string file
variable = sharedPreferences.getBoolean(getString(R.string.pref_parameter_key), getString(R.string.pref_parameter_value));
// pref_parameter_value are the default values
```
The problem with this method is that the method is in the ```onCreate``` method, that is only called when the app launches, rotates...

#### A better way 
1. Make the the MainActivity implementing ```SharedPreferences.OnSharedPreferenceChangeListener```, overrided ```onSharedPreferenceChanged```, with something like the code below:
```java
public void onSharedPreferenceChanged(SharedPreferences sharedPreferences, String key){
	if(key.equals(getString(R.string.pref_parameter_key)){
	// ...
	}
}
```
2. Register the listener by calling ```sharedPreferences.registerOnSharedPrefernceChangerListener(this)``` in the onCreate method.
3. Unregister the method by overriding the ```onDestroy``` method:
```PreferenceManager.getDefaultSharedPreferences(this).unregisterONSharedPreferenceChangeListener(this)```.

### Other Preferences
* ListPreference
An example:
```xml

```
* TextPreference
An example:
```xml
    <EditTextPreference
        android:defaultValue="Mountain View, CA 94843"
        android:inputType="text"
        android:key="location"
        android:singleLine="true"
        android:title="Location"
        >
    </EditTextPreference>
```
##### Notes
* __TextPreference check input__ :For TextPreference, the user can enter everything. Hence one should restrict and control what the user has inputed and act consequently.
To do so, you'll have to make the SettingsActivity inherited from ```Preference.OnPreferenceChangeListener```. Instead of ```OnSharedPreferenceChangeListener```, it is triggered BEFORE any value is saved in the SharedPreference file. The method to override is ```onPreferenceChange```. The listener should be attach to the  preference in onCreatePreferences:
```java 
public void onCreatePreferences(Bundle bundle, String s) {
         /* Other preference setup code code */
        //...
        Preference preference = findPreference(getString(R.string.pref_size_key));
        preference.setOnPreferenceChangeListener(this);
}
```
### Should I create a settings
![](https://d17h27t6h515a5.cloudfront.net/topher/2016/November/582a4eda_screen-shot-2016-11-14-at-3.55.51-pm/screen-shot-2016-11-14-at-3.55.51-pm.png)

# SQLite
### How to create a database
* Create a class that will represent a Table and its entries in a database:
```java
public final class TableClass{
	// constructor is private because it should not be used
	private TableClass() {}

	public static class TableClass implements BaseColumns{
		public static final String TABLE_NAME = "";
		public static final String COLUMN_NAME_WHATEVER = "whatever";
		...
	
	}
}
```
* Create the DataBase:
```java
public class DataBaseHelper extends SQLiteOpenHelper {
	// name of the database
	private static final String databaseName = "nameDB.db";
	// version of the database
	private static final int database_version = 1;

	// constructor
	public DataBaseHelper(Context context) {
		// just call parent constructor		
		super(context, DATABASE_NAME, null, DATABASE_VERSION);
	}

	// method that will actually create the database file
	public onCreate(SQLiteDatabase sqliteDatabase) {
		final String SQL_CREATE_DATABASE = "CREATE TABLE " + TableClass.TABLE_NAME + " (" + TableClass._ID + "INTEGER PRIMARY KEY AUTOINCREMENT," + TableClass.COLUMN_NAME_WHATEVER + "STRING NOT NULL" + ... + ");";
```	sqliteDatabase.execSQL(SQL_CREATE_DATABASE);
	}
}
```
### Insert and retrieve from the Database in an Activity
* Create a private parameter ```private SQLiteDatabase database;```, 
* Get the database with the command:
```java
WaitlistDbHelper wldbH = new WaitlistDbHelper(this);
database = wldbH.getWritableDatabase();
``` 
_Note that getReadableDatabase return a read-only database object._
* Query a database with the command
```sqliteDatabase.query()```. Here is the order of the arguments: location of the database/table name, filter of the columns, statement for how to filter rows, what to filter, sort order to return data)
* Insert in a database by creating a ContentValues:
```java
ContentValues contentValues = new ContentValues();
contentValues.put(KEY_NAME, value);
contentValues.put(KEY_NAME2, value2);
database.insert(TABLE_NAME, null, contentValues);
```
* Remove from the database (see the documentation).

# Content Provider
Every android developer should be aware of 4 key app components: 
* Activities
* Services
* Broadcast receivers
* Content provider
### Use Content provider from others
1. To have the access to read to a content provider, a permission must be added to the AndroidManifest:
```
<uses-permission android:name="com.example.udacity.application.TERMS_READ" />
```

2. To get acess to a content provider, one must create a ```ContentResolver``` from the ```getContentResolver```, which can perform four operations: update, query, insert, delete. To specify which contentProvider to get acess to, one must use an URI. An URI is of the form:
```ContentProviderPrefix://ContextAuthority/SpecificData```.
3. Here are some methods to use a Cursor and extract information from it:
* .getColumnIndex(COLUMN_NAME): return the index of the column
* .moveToNext() : move the pointer of the cursor to the next row, and return false when there are no more rows.
* .getString(Index): return the value at the cursor row pointer for the Index column 
* .close(): always close a cursor at the end

### Create a Content provider
* Create a class MyContentProvider that inherited from ContentProvider 
* Register the ContentProvider in the Manifest so as to be see by the system.
```xml
<provider
	android:authorities="package.name"
	android;name="package.name + class.name"
	android:exported="true if every app can access the CP"
/>
```
* MyContentProvider must override onCreate, delete, query... A URI is passed for query, delete, insert method, so the method inside must deal with different URI format. One way to deal with it is to use UriMatcher. A UriMatcher helps deals with different targeted path data. For example one may want to access a all table or only a single row refered by its ID (an Integer). So the URI must reflect this need. One example of an URI for this problem is contextauthority/tableName/#. '#' means that every integer could be pass, '*' means every sort of string.  
When the UriMatcher is build, it takes an URI, and return a different code for every different type handle for the URI. Here is how you can build an UriMatcher:
```java
URIMatcher buildUriMatcher(){
	// create a empty uriMatcher
	UriMatcher uriMatcher = new UriMatcher(UriMatcher.NO_MATCH);
	// 1. AUTHORITIES is in ContentProvider class	
	// 2. '/#': match every URI with an integer at the end
	// 3. The code can be used as a switch in query/delete... functions where different treatment should be applied to different URI. See the example below
uriMatcher.addURI(MyContentProvider.AUTHORITIES, table.path + "/#',ID_to_return); 
}
```

Here is how you can use the UriMatcher in the insert function for example:
```
Uri insert(Uri uri, ContentValues values) {
	final SQLiteDataBase db = mTaskDbHelper.getWritableDatabase();
	int match = mUriMatcher.match(uri);
	switch(match){ ...
}

# Background task
### Services
No networking activity should be done in an Activity because we can leave the activity. This is what Services is for. It is meant for running background tasks that don't need a visual component
#### Loaders versus Services
* If you're loading or processing data that will be used in the UI, use a loader
* If the process is decoupled from the user interface, and exists even if there is no user interface, then a Service should be used. 
### Starting services
Services that are calling and started immediately.
```java
Intent intent = new Intent(this, MyIntentService.class);
startService(intent);
```
MyIntentService must override ```onHandleIntent(Intent intent)```.
Here is how one can implement a IntentService.
```java
public class MyIntentService extends IntentService {
	public MyIntentService() {super("MyIntentService"); }
	public void onHandleIntent(Intent intent) {
		// get the action that we should executed (set with ... setAction() :)
		String action = intent.getAction();
		...		
	}

}
* Every service should be defined in the Manifest.xml
* Notes that every intent services are run in another thread than the main one, but there is only one background thread for every intent. Hence every intent of the app are put in a queue, and done in order of call.

### Pending intents and Notifications
A ```PendingIntent``` is a token that you give to a foreign application (e.g. NotificationManager, AlarmManager, Home Screen AppWidgetManager, or other 3rd party applications), which allows the foreign application to use your application's permissions to execute a predefined piece of code.
#### Create a New notification when clicking on a button
1. Create a Pending Intent
```java
Intent intent = new Intent(context, MainActivity.class);
PendingIntent pi = PendingIntent.getActivity(context,
                ID_PENDING_INTENT,
                intent,
                PendingIntent.FLAG_UPDATE_CURRENT);
```
* 2. Using ```NotificationBuilder``` create a Notification:
```java
NotificationCompat.Builder notificationBuilder = new NotificationCompat.Builder(context)
        .setColor(ContextCompat.getColor(context, R.color.colorPrimaryDark))
	.setSmallIcon(R.drawable...)
	.setContentTitle(context.getString(R.string...))
	.setContentText(context.getString(R.string...))
	.setDefaults(NotificationCompat.DEFAULT_VIBRATE)
	.setContentIntent(contentIntent(context))
	.setAutoCancel(true);
```
* 3. It is possible to display to set the priority of the Notification by using a function only available in the recent JDK. Here the example also show how to adapt the code for different JDK:
```java
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN) {
   notificationBuilder.setPriority(NotificationCompat.PRIORITY_HIGH);
}
```
* 4. Launch the notification using a ```NotificationManager```:
```java
NotificationManager notificationManager = (NotificationManager) context.getSystemService(context.NOTIFICATION_SERVICE);
notificationManager.notify(ID_NOTIFICATION, notificationBuilder.build());
```
### Add an action for the Notification
1. First you need to create an Action:
```java
// Create an intent
Intent intent = new Intent(context, SERVICES_or_ACTIVITIES.class);
intent.setAction(ID_ACTION);
// Create a PendingIntent (here its a Service that is launches
PendingIntent pendingIntent = PendingIntent.getService(context,
                PENDING_INTENT_ID,
                intent,
                PendingIntent.FLAG_UPDATE_CURRENT);
// Create an Action
NotificationCompat.Action action = new NotificationCompat.Action(R.drawable....,
                "Message",
                pendingIntent);
```
2. Then you need to add the action with the ```addAction``` on the ```NotificationCompat.Builder```.

### Create a notification at a specific posterior time
For example, one might want an app that pop a notification every 15 minutes or so. 
1. Create a method (in a new class for example) that will use ```FirebaseJobDispatcher``` to schedule a job that repeats roughly:
```java
public static void method(Context context) {
	// check first if the job hasn't been already started by creating a boolean in the class
		
	// Create a new GooglePlayDriver
	Driver driver = new GooglePlayDriver(context);

	// Create a new FirebaseJobDispatcher
	FirebaseJobDispatcher dispatcher = new FirebaseJobDispatcher(context);

	Job constraintReminderJob = dispatcher.newJobBuilder()
		// add the service (created in 2.)		
	.setService(JobService.class)
	.setTag(REMINDER_JOB_TAG)
		// Add constraints (for example, the device must be charging
	.setConstraints(Constraint.DEVICE_CHARGING)
		// How long this job should persist
	.setLifetime(Lifetime.FOREVER)
	.setRecurring(true)
		// When to trigger the job (see documentation)
	.setTrigger()
		// If another job is created with the same tag, just replace the old one
	.setReplaceCurrent(true)
		// build it
	.build();
	
	dispatcher.schedule(constraintReminderJob);
}
```
2. Then, you need to create a MyJobService class extending JobService:
```java
class MyJobService extends JobService {
	// override onStartJob(final JobParameters jobParameters) {}
	// The entry point to your Job. Implementations should offload work to another thread of execution as soon as possible.
	
	// override onStopJob(JobParameters jobParameters) {}
	// Called when the scheduling engine has decided to interrupt the execution of a running job,
     	// most likely because the runtime constraints associated with the job are no longer satisfied.
}

##### Notes
* To reset all notifications, you can use the method ```NotificationManager.cancelAll()```.


### Broadcast Receiver
A broadcast Receiver is a listener created to catch some intent from the system or other apps, that are used for the App. For example, when a photo is taken
#### Static receiver
A static receiver listens even if your app is not launched. It mush be defined in the Manifest
```xml
<receiver android:name=".MyBroadcastReceiver">
	<intent-filter>
		<action android:name="com.android.camera.NEW_PICTURE" />
		<action android:name="android.hardware.action.NEW_PICTURE" />
		<data android:mimeType="image/*" />
	</intent-filter>
</receiver>
```

#### Dynamic receiver
A dynamic receiver is link to the state of the app, and is created intent receives are __only__ usefull when the app is running
* 1. Create an Intent filter and set which intent should we received
```java
IntentFilter intentFilter = new IntentFilter();
intentFilter.addAction(Intent.ACTION_POWER_CONNECTED);
```
2. Create a ```BroadCastReceiver``` class and override ```onReceive()```
```java
class MyBroadCastReceiver extends BroadcastReceiver{
	public void onReceive(Context context, Intent intent) {
		String action = intent.getAction();
		if(action == Intent.ACTION_POWER_CONNECTED) {
			// do something
		}
	}
}
```
3. Because the receiver is only triggered when the app is launched, you need to register ```registerReceiver()``` and unregister ```unregisterReceiver()``` in the ```onCreate()```, ```onDestroy()```... methods

# Completing the UI
### Constraint layout
Dependencies in the build.gradle for the Constraint Layout library ```compile 'com.android.support.constraint:constraint-layout:1.0.0-beta4'```  
No particular information about this layout, I used the Designer mode
### Accessibility
* You make your app accessible for people with disabilities. For example every element in the screen can be represented as a string, which will be pronounced by the Android phone. You just have to set the ```android:contentDescription``` field for every element in the xml.
* __sp sizes__ are invariant whether the density of the pixel of the device. It also allow visually impared people to increase the size of the text in their Accessibility parameter.
* Add color in the color folder. Always defines ```android:colorPrimary```, ```android:colorPrimaryDark```, and ```android:colorAccent```.
### Style and Style inheritance

If different item have the same style, then its better to define a Style in the style.xml file and instead of defining every parameter for every items, just set one: ```style="@style/myStyle"```. Here is a simple Style
```xml
<style name="myStyle">
	<item name="android:textSize">28sp</item>
</style>
```
* Style inheritance: ```<style name="myStyle" parent="mainStyle">```. Then, it is possible to add a new parameter, or even override one from the parent.

### Responsive design
It is a design that respond
