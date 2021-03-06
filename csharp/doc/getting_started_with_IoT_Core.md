# Getting Started with Azure IoT Hub on Windows 10 IoT Core

## Objective

This step-by-step guide will allow you to familiarize yourself with Windows 10 IoT Core platform, set up your device and create your first application that connects to Azure IoT Hub.

## Windows 10 IoT Core

You can leverage the power of the Windows platform and Visual Studio to create innovative solutions on a variety of devices running Windows 10 IoT Core.

## Setting Up You Device

First of all, you need to set up your device. 

- If you’re using Raspberry Pi, set up your device according to instructions [here](http://ms-iot.github.io/content/en-US/win10/SetupRPI.htm).
- If you’re using MinnowBoard Max set up your device according to instructions [here](http://ms-iot.github.io/content/en-US/win10/SetupMBM.htm).


## Sign Up To Azure IoT Hub

Follow the instructions [here](https://account.windowsazure.com/signup?offer=ms-azr-0044p) on how to sign up to the Azure IoT Hub service.

As part of the sing up process, you will receive the connection string. You will need to use it later for connecting to the service.

## Install Visual Studio 2015 and Tools

To create Windows IoT Core solutions, you will need to install [Visual Studio 2015](https://www.visualstudio.com/en-us/products/vs-2015-product-editions.aspx). You can install any edition of Visual Studio, including the free Community edition.

Make sure to select the "Universal Windows App Development Tools", the component required for writing apps Windows 10:

![Universal Windows App Development Tools](pngs\install_tools_for_windows10.png)

## Create a Visual Studio UWP Solution

UWP (Universal Windows Platform) is an evolution of Windows application model introduced in Windows 8. UWP provides a common app platform available on every device that runs Windows 10, including the IoT Core.

To create a UWP solution in Visual Studio, select File -> New -> Project:

![New Project Creation](pngs\new_project_menu.png)

In the New Project dialog that comes up, select "Blank App (Universal Windows) Visual C#. Give your project a name, for example "MyFirstIoTCoreApp": 

![New Solution Dialog](pngs\new_solution.PNG)

## Get Microsoft.Azure.Devices.Client Library

Once the project has been created, we need to include a reference to the Microsoft.Azure.Devices.Client library -- this is the component that contains the functionality necessary to connect to Azure IoT Hub.

In the Solution Explorer window, right-click on the solution and select "Managed NuGet Packages":

![Manage NuGet Packages...](pngs\manage_NuGet.png)

The NuGet Package Manager will open. Find the package named "Microsoft.Azure.Devices.Client" and click Install:  

![Installing Microsoft.Azure.Devices.Client package](pngs\find_madc.png)

The package will be downloaded and added to your project.

## Build and run the Device Explorer tool

You can use the Device Explorer sample application on your Windows desktop machine to create and register a device ID and symmetric key for your device. The Device Explorer interfaces with Azure IoT Hubs, and has some basic capabilities such as:

- **Device management**: creates device IDs and obtains a list of registered devices on your IoT Hub.
- **Monitors and consumes data** sent by your devices to your IoT Hub.
- **Sends messages** to your devices.

To run this tool, you need connection and configuration information for your IoT Hub.

- Clone or download a copy of the [azure-iot-suite-sdks](https://github.com/Azure/azure-iot-suite-sdks) GitHub repository on your Windows desktop machine.
- In Visual Studio, open the DeviceExplorer.sln solution in the tools\\DeviceExplorer folder in your local copy of the repository, then build and run it.
- Paste the connection string for your IoT Hub and then click **Update**.
- Click the **Management** tab, then create a device ID for your device and register your device with your IoT Hub by clicking the **Create** button and entering a device name.

## Sample 1: Sending Data to Azure IoT Hub

At last, you can start writing code! Open the file named `MainPage.xampl.cs` and add a few `using` directives:

    using System.Text;
    using System.Threading.Tasks;
    using Microsoft.Azure.Devices.Client;

In our first sample, we will try to send a simple string to Azure IoT Hub. 

Add the following function to class `MainPage`:

    private async Task SendDataToAzure()
    {
        DeviceClient deviceClient = DeviceClient.CreateFromConnectionString("<replace>", TransportType.Http1);
    
        var text = "Hellow, Windows 10!";
        var msg = new Message(Encoding.UTF8.GetBytes(text));
    
        await deviceClient.SendEventAsync(msg);
    }

### Using The Real Connection String

Important: remember to replace the `"<replace>"` string in the above snippet with the actual connection string. To get the connection string for your device, return the Device Explorer's **Management** tab, find your device in the list, right-click on it and select "Copy connection string for selected device". Now paste this string into your code, replacing the string `"<replace>"`.

This string gives you access to the service, so remember to keep it secret -- in particular, you don't want to check it into your source control system, especially if you project is open source.

### Build and Run

Finally, it's time to run the app! Add the call to the function `SendDataToAzure` from the constructor of the `MainPage` class:

![](pngs\invoke_function.png)

Now choose the right architecture (x86 or ARM, depending on your device) and set the debugging method to "Remote Machine":

![](pngs\select_arch.png)

If you're using a Raspberry Pi device, select ARM. For other supported devices, consult [windowsondevices.com](http://www.windowsondevices.com). 

You will need to deploy your app to an IoT device so that you can run and debug it remotely. Right-click on the solution in the Solution Explorer, select Properties and navigate to the Debug tab:

![](pngs\select_device.png)

Type in the name of your device. Make sure the "Use authentication" box is unchecked.

Now, you're ready to build and deploy the app. Choose "Build" and then "Deploy" from the top menu.

Hit F5 to run the app. It might take a few second for the app to start.  

## Validate Your Solution

Use the Device Explorer to inspect data coming to Azure IoT Hub and manage devices.

Navigate to the Data tab in the Device Explorer and make sure you see "Hello, Windows 10" string there:

![](pngs\device_explorer_data.png)

Congratulations! You have successfully sent your string to Azure IoT Hub.

## Sample 2: Receiving Data from Azure IoT Hub

In this sample, we will learn how to receive data from Azure IoT Hub. Define the function `ReceiveDataFromAzure` as follows:

    public async static Task ReceiveDataFromAzure()
    {
        DeviceClient deviceClient = DeviceClient.CreateFromConnectionString("<replace>", TransportType.Http1);
    
        Message receivedMessage;
        string messageData;
    
        while (true)
        {
            receivedMessage = await deviceClient.ReceiveAsync();
    
            if (receivedMessage != null)
            {
                messageData = Encoding.ASCII.GetString(receivedMessage.GetBytes());
                await deviceClient.CompleteAsync(receivedMessage);
            }
        }
    }

This function can be invoked from `MainPage` similarly to how we did in the previous example.

    public MainPage()
    {
        this.InitializeComponent();
        ReceiveDataFromAzure();
    }

Now, before you run the app, switch to the Device Explorer and open the Notification tab:

![](pngs\device_explorer_notif.png)

Select your device in the Device ID list, and type "hello, device!" in the Notification box. Hit Send.

Now set the breakpoint at the `await` statement and start the application. When the control reaches the breakpoint, you should see the value of `messageData` as "hello, device!":

![](pngs\hello_device_received.png)

Congratulations! You have successfully received your string from Azure IoT Hub.