# Synchronization Management


You can manage a synchronization schedule for applications by using a UI application to request for sync jobs through the Sync Manager, and a service application to listen for the requests through the Sync Adapter. The service and UI applications must have the same package name.

The main features of the Tizen.Account.SyncManager namespace are:

-   Sync Adapter
    -   Setting Sync Adapter callbacks

        You can [set the callback methods](#set_callback) in your Sync Adapter service application that your client application can call to set up a sync operation.

-   Sync Manager
    -   Defining a sync job

        You can [create a new sync job](#set_parameters) and add user data into it as either an account information instance or a data bundle.

    -   Requesting an on-demand sync job

        You can [add an on-demand sync job](#on_demand_sync) for a one-time operation.

    -   Requesting a periodic sync job

        You can [add a periodic sync job](#periodic_sync) with a recurring cycle.

    -   Requesting a data change sync job

        You can [add a data change sync job](#data_change_sync) for receiving a notification whenever a specific database changes.

    -   Retrieving all registered sync jobs

        You can [get a list of all registered sync jobs](#foreach_sync).

    -   Removing sync jobs

        You can [remove registered sync jobs](#remove_sync) when they are no longer needed.

## Prerequisites


To enable your application to use the synchronization management functionality, follow the steps below:

1.  To use the [Tizen.Account.SyncManager](/application/dotnet/api/TizenFX/latest/api/Tizen.Account.SyncManager.html) namespace, the application has to request permission by adding the following privileges to the `tizen-manifest.xml` file:

    ```XML
    <privileges>
       <privilege>http://tizen.org/privilege/account.read</privilege>
       <privilege>http://tizen.org/privilege/account.write</privilege>
       <privilege>http://tizen.org/privilege/alarm.set</privilege>
       <privilege>http://tizen.org/privilege/calendar.read</privilege>
       <privilege>http://tizen.org/privilege/contact.read</privilege>
    </privileges>
    ```

2.  To use the methods and properties of the `Tizen.Account.SyncManager` namespace, include it in your application:

    ```csharp
    using Tizen.Account.SyncManager;
    ```

> [!NOTE]
> To use the features of the [Tizen.Account.SyncManager](/application/dotnet/api/TizenFX/latest/api/Tizen.Account.SyncManager.html) namespace, the service application must first [set the callbacks](#set_callback). A UI application cannot initialize or set callback methods through the [Tizen.Account.SyncManager.SyncAdapter](/application/dotnet/api/TizenFX/latest/api/Tizen.Account.SyncManager.SyncAdapter.html) class. Instead, the UI application must call the methods of the [Tizen.Account.SyncManager.SyncClient](/application/dotnet/api/TizenFX/latest/api/Tizen.Account.SyncManager.SyncClient.html) class to request sync operations from the service application.

<a name="set_callback"></a>
## Set sync adapter callbacks

To set callbacks in your Sync Adapter service application that your UI application can call to request sync operations:

1.  Set up event handlers for starting and stopping data synchronization. When the `StartSyncCallback()` callback of the [Tizen.Account.SyncManager.SyncAdapter](/application/dotnet/api/TizenFX/latest/api/Tizen.Account.SyncManager.SyncAdapter.html) class is invoked, the predefined data sync process is performed inside the callback method. The `CancelSyncCallback()` callback works in a similar way and cancels the data sync process:

    ```csharp
    static bool StartSyncCallback(SyncJobData data)
    {
        /// Code for starting data synchronization

        return true;
    }

    static void CancelSyncCallback(SyncJobData data)
    {
        /// Code for cancelling data synchronization
    }
    ```

2.  Register the event callbacks with the `SetSyncEventCallbacks()` method of the `Tizen.Account.SyncManager.SyncAdapter` class to receive notifications regarding the sync operation.

    When a specific event is detected or a sync job is requested, the `StartSyncCallback()` or `CancelSyncCallback()` callbacks are invoked:

    ```csharp
    SyncAdapter obj = new SyncAdapter();
    obj.SetSyncEventCallbacks(StartSyncCallback, CancelSyncCallback);
    ```

3.  When the sync operations are no longer needed, unset the callbacks to free the `Tizen.Account.SyncManager.SyncAdapter` instance:

    ```csharp
    obj.UnsetSyncEventCallbacks();
    ```

<a name="set_parameters"></a>
## Define sync job

To define a sync job, create a new [Tizen.Account.SyncManager.SyncJobData](/application/dotnet/api/TizenFX/latest/api/Tizen.Account.SyncManager.SyncJobData.html) instance:

```csharp
SyncJobData request = new SyncJobData();
request.SyncJobName = "PeriodicSyncJob";
```

You can add user data to a sync job as an account information instance or as a data bundle, follow these steps to add user data as an account information or as a data bundle:

-   To add account information to a sync job, create a new instance of the [Tizen.Account.AccountManager.Account](/application/dotnet/api/TizenFX/latest/api/Tizen.Account.AccountManager.Account.html) class. Add your account information to it, and then add it into the sync job as the `Account` property of the `Tizen.Account.SyncManager.SyncJobData` instance. For more information about creating accounts, see [Creating and Managing an Account](account.md#add):

    ```csharp
    using Tizen.Account.AccountManager;

    AccountManager.Account account = AccountManager.Account.CreateAccount();
    account.UserName = "Kim";
    account.SyncState = AccountSyncState.Idle;
    AccountService.AddAccount(account);

    SyncJobData request = new SyncJobData();
    request.Account = account;
    ```

-   To add a data bundle to a sync job, create a new instance of the [Tizen.Applications.Bundle](/application/dotnet/api/TizenFX/latest/api/Tizen.Applications.Bundle.html) class, add your data to it, and add it as the `UserData` property of the `Tizen.Account.SyncManager.SyncJobData` instance:

    ```csharp
    using Tizen.Applications;

    Applications.Bundle bundle = new Applications.Bundle();
    bundle.AddItem("key", "value");

    SyncJobData request = new SyncJobData();
    request.UserData = bundle;
    ```

<a name="on_demand_sync"></a>
## Request on-demand sync job

To request a one-time sync job from the Sync Adapter service application, use `RequestOnDemandSyncJob()` of the [Tizen.Account.SyncManager.SyncClient](/application/dotnet/api/TizenFX/latest/api/Tizen.Account.SyncManager.SyncClient.html) class:

```csharp
SyncJobData request = new SyncJobData();
request.SyncJobName = "OnDemandSyncJob";
int id = SyncClient.RequestOnDemandSyncJob(request, SyncOption.NoRetry);
```

<a name="periodic_sync"></a>
## Request a periodic sync job

To register a periodically-recurring sync operation with the Sync Adapter service application, follow these steps:

-   To set up a periodic sync job with a regular sync interval, use `AddPeriodicSyncJob()` of the [Tizen.Account.SyncManager.SyncClient](/application/dotnet/api/TizenFX/latest/api/Tizen.Account.SyncManager.SyncClient.html) class, and give the sync interval as a value of the [Tizen.Account.SyncManager.SyncPeriod](/application/dotnet/api/TizenFX/latest/api/Tizen.Account.SyncManager.SyncPeriod.html) enumeration. In the following example, the sync interval is set to 30 minutes:

    ```csharp
    SyncJobData request = new SyncJobData();
    request.SyncJobName = "PeriodicSyncJob";
    int id = SyncClient.AddPeriodicSyncJob(request, SyncPeriod.ThirtyMin, SyncOption.None);
    ```

    You can also add additional parameters to the sync job using values of the [Tizen.Account.SyncManager.SyncOption](/application/dotnet/api/TizenFX/latest/api/Tizen.Account.SyncManager.SyncOption.html) enumeration. The value `NoRetry` means that the application does not retry the sync job if it fails, and `Expedited` means that the sync job is handled as soon as possible:

    ```csharp
    id = SyncClient.AddPeriodicSyncJob(request, SyncPeriod.OneHour, SyncOption.NoRetry);
    id = SyncClient.AddPeriodicSyncJob(request, SyncPeriod.OneDay, SyncOption.Expedited);
    ```

-   You can renew a previously registered periodic sync job with `AddPeriodicSyncJob()` by using the same [Tizen.Account.SyncManager.SyncJobData](/application/dotnet/api/TizenFX/latest/api/Tizen.Account.SyncManager.SyncJobData.html) instance with new parameters:

    ```csharp
    SyncJobData request = new SyncJobData();
    request.SyncJobName = "PeriodicSyncJob";
    int id = SyncClient.AddPeriodicSyncJob(request, SyncPeriod.ThirtyMin, SyncOption.None);
    id = SyncClient.AddPeriodicSyncJob(request, SyncPeriod.TwoHours, SyncOption.Expedited);
    ```

<a name="data_change_sync"></a>
## Define a data change sync job

To register a data change sync job with the Sync Adapter service application, to occur whenever corresponding data changes, follow these steps:

-   Add a data change sync job with `AddDataChangeSyncJob()` of the [Tizen.Account.SyncManager.SyncClient](/application/dotnet/api/TizenFX/latest/api/Tizen.Account.SyncManager.SyncClient.html) class. This method adds the sync job only for the capability given as the value of the `SyncJobName` property of the [Tizen.Account.SyncManager.SyncJobData](/application/dotnet/api/TizenFX/latest/api/Tizen.Account.SyncManager.SyncJobData.html) instance. For example, to add a data change sync job for the calendar:

    ```csharp
    SyncJobData request = new SyncJobData();
    request.SyncJobName = SyncJobData.CalendarCapability;
    int id = SyncClient.AddDataChangeSyncJob(request, SyncOption.None);
    ```

    You can also add additional parameters to the sync job using values of the [Tizen.Account.SyncManager.SyncOption](/application/dotnet/api/TizenFX/latest/api/Tizen.Account.SyncManager.SyncOption.html) enumeration. The value `NoRetry` means that the application does not retry the sync job if it fails, and `Expedited` means that another sync job is handled as soon as possible:

    ```csharp
    SyncJobData request2 = new SyncJobData();
    request2.SyncJobName = SyncJobData.ContactCapability;
    SyncJobData request3 = new SyncJobData();
    request3.SyncJobName = SyncJobData.ImageCapability;
    int id2 = SyncClient.AddDataChangeSyncJob(request2, SyncOption.NoRetry);
    int id3 = SyncClient.AddDataChangeSyncJob(request3, SyncOption.Expedited);
    ```

-   You can renew a previously registered data change sync job with the `AddDataChangeSyncJob()` method by using the same `Tizen.Account.SyncManager.SyncJobData` instance with new parameters:

    ```csharp
    SyncJobData request = new SyncJobData();
    request.SyncJobName = SyncJobData.ContactCapability;
    int id = SyncClient.AddDataChangeSyncJob(request, SyncOption.Expedited);
    id = SyncClient.AddDataChangeSyncJob(request, SyncOption.None);
    ```

<a name="foreach_sync"></a>
## Retrieve all registered sync jobs

To retrieve a list of all registered sync jobs, use the `GetAllSyncJobs()` method of the [Tizen.Account.SyncManager.SyncClient](/application/dotnet/api/TizenFX/latest/api/Tizen.Account.SyncManager.SyncClient.html) class:

```csharp
SyncJobData request = new SyncJobData()
{
    SyncJobName = "PeriodicSyncJob"
};
int periodicId = SyncClient.AddPeriodicSyncJob(request, SyncPeriod.ThreeHours, SyncOption.None);

SyncJobData request2 = new SyncJobData()
{
    SyncJobName = SyncJobData.MusicCapability
};
int dataChangeId = SyncClient.AddDataChangeSyncJob(request2, SyncOption.None);

IEnumerable<KeyValuePair<int, SyncJobData>> syncJobs = SyncClient.GetAllSyncJobs();
foreach (KeyValuePair<int, SyncJobData> item in syncJobs)
{
    if (item.Key == periodicId)
    {
        Console.WriteLine(item.Value.SyncJobName.ToString());
    }
    if (item.Key == datachangeId)
    {
        Console.WriteLine(item.Value.SyncJobName.ToString());
    }
}
```

<a name="remove_sync"></a>
## Remove sync jobs

To remove registered sync jobs, use the `RemoveSyncJob()` method of the [Tizen.Account.SyncManager.SyncClient](/application/dotnet/api/TizenFX/latest/api/Tizen.Account.SyncManager.SyncClient.html) class, using the `id` property of the job to be removed:

```csharp
SyncJobData request = new SyncJobData();
request.SyncJobName = "PeriodicSyncJob";

SyncJobData request2 = new SyncJobData();
request2.SyncJobName = SyncJobData.ImageCapability;

int id = SyncClient.AddPeriodicSyncJob(request, SyncPeriod.OneHour, SyncOption.Expedited);
int id2 = SyncClient.AddDataChangeSyncJob(request2, SyncOption.None);

SyncClient.RemoveSyncJob(id);
SyncClient.RemoveSyncJob(id2);
```


## Related information
* Dependencies
  -   Tizen 4.0 and Higher
