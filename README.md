
# BACnet4J Docs

## Common Terms
1. **LocalDevice:**
   The `LocalDevice` class represents a BACnet device within your Java application. It encapsulates the functionality related to managing local device properties, handling incoming requests from other devices on the BACnet network, and sending requests to remote devices. This class is crucial for initializing and managing communication with BACnet devices locally.

```java 
import com.serotonin.bacnet4j.LocalDevice;
import com.serotonin.bacnet4j.exception.BACnetException;
import com.serotonin.bacnet4j.type.constructed.PropertyValue;
import com.serotonin.bacnet4j.type.enumerated.PropertyIdentifier;

public class BacnetLocalDeviceExample {
    public static void main(String[] args) {
        try {
            // Create a local BACnet device with device ID 1234
            LocalDevice localDevice = new LocalDevice(1234);
            localDevice.initialize();

            // Read a property from a remote BACnet device (device ID 5678)
            PropertyIdentifier propertyId = PropertyIdentifier.PRESENT_VALUE;
            PropertyValue propertyValue = localDevice.readProperty(5678, propertyId);

            System.out.println("Value of " + propertyId + " on device 5678: " + propertyValue);

            // Clean up resources
            localDevice.terminate();
        } catch (BACnetException e) {
            e.printStackTrace();
        }
    }
}


```

2. **NotificationClassListener:**
   The `NotificationClassListener` interface is likely used for handling notifications or alarms sent by BACnet devices. In a building automation system, devices can send notifications regarding events such as temperature exceeding a threshold, equipment failures, etc. Implementing this interface allows your application to react to these notifications and take appropriate actions.

```java
import com.serotonin.bacnet4j.event.DeviceEventAdapter;
import com.serotonin.bacnet4j.event.DeviceEventProducer;
import com.serotonin.bacnet4j.type.Encodable;
import com.serotonin.bacnet4j.type.enumerated.PropertyIdentifier;

public class BacnetNotificationListenerExample extends DeviceEventAdapter {
    @Override
    public void eventReceived(DeviceEventProducer dep, Encodable newValue) {
        if (newValue instanceof PropertyIdentifier) {
            PropertyIdentifier propertyId = (PropertyIdentifier) newValue;
            System.out.println("Notification received for property: " + propertyId);
            // Add your handling code here
        }
    }

    public static void main(String[] args) {
        // Initialize BACnet notification listener
        BacnetNotificationListenerExample listener = new BacnetNotificationListenerExample();

        // Add the listener to your LocalDevice
        LocalDevice localDevice = new LocalDevice(1234);
        localDevice.getEventHandler().addListener(listener);

        // Start listening for notifications
        localDevice.initialize();
    }
}
```
3. **RemoteDevice:**
   The `RemoteDevice` class represents a remote BACnet device that your application communicates with over the BACnet network. It encapsulates information about the remote device, such as its device ID, network address, supported services, etc. This class is used when establishing connections, sending requests, and receiving responses from remote devices.

```java
import com.serotonin.bacnet4j.RemoteDevice;
import com.serotonin.bacnet4j.type.enumerated.ObjectType;

public class BacnetRemoteDeviceExample {
    public static void main(String[] args) {
        // Create a remote BACnet device with device ID 5678 and instance number 1
        RemoteDevice remoteDevice = new RemoteDevice(5678, 1, "Remote Device", ObjectType.analogValue);

        // Access remote device properties
        int deviceId = remoteDevice.getId();
        int instanceNumber = remoteDevice.getInstanceNumber();
        String deviceName = remoteDevice.getName();
        ObjectType objectType = remoteDevice.getObjectType();

        System.out.println("Remote Device ID: " + deviceId);
        System.out.println("Remote Device Instance Number: " + instanceNumber);
        System.out.println("Remote Device Name: " + deviceName);
        System.out.println("Remote Device Object Type: " + objectType);
    }
}
```

4. **RemoteObject:**
   The `RemoteObject` class represents a BACnet object on a remote device. BACnet objects can include things like temperature sensors, HVAC controllers, lighting controls, etc. This class likely provides methods for reading and writing properties of remote objects, subscribing to change notifications (COV), and performing other object-specific operations.

```java
import com.serotonin.bacnet4j.RemoteObject;
import com.serotonin.bacnet4j.exception.PropertyValueException;
import com.serotonin.bacnet4j.type.constructed.PropertyValue;
import com.serotonin.bacnet4j.type.enumerated.PropertyIdentifier;
import com.serotonin.bacnet4j.type.primitive.Real;

public class BacnetRemoteObjectExample {
    public static void main(String[] args) {
        // Create a remote object (e.g., Analog Value) with object ID 1234 and instance number 1
        RemoteObject remoteObject = new RemoteObject(1234, 1);

        try {
            // Read a property (e.g., Present Value) from the remote object
            PropertyIdentifier propertyId = PropertyIdentifier.PRESENT_VALUE;
            PropertyValue propertyValue = remoteObject.readProperty(propertyId);
            System.out.println("Value of " + propertyId + ": " + propertyValue);

            // Set a new value for the property (e.g., Present Value)
            Real newValue = new Real(25.0);
            remoteObject.writeProperty(new PropertyValue(propertyId, newValue));

            // Clean up resources
            remoteObject.terminate();
        } catch (PropertyValueException e) {
            e.printStackTrace();
        }
    }
}
```

5. **ResponseConsumer:**
   The `ResponseConsumer` interface or class is likely used for handling responses received from BACnet devices after sending requests. It defines methods or callbacks for processing responses, extracting data, and handling error conditions. Implementing or using `ResponseConsumer` allows your application to handle asynchronous responses in a structured manner.

```java
import com.serotonin.bacnet4j.exception.BACnetException;
import com.serotonin.bacnet4j.service.unconfirmed.WhoIsRequest;
import com.serotonin.bacnet4j.transport.DefaultTransport;
import com.serotonin.bacnet4j.transport.Transport;
import com.serotonin.bacnet4j.type.constructed.SequenceOf;
import com.serotonin.bacnet4j.type.enumerated.PropertyIdentifier;
import com.serotonin.bacnet4j.type.primitive.ObjectIdentifier;

public class BacnetResponseConsumerExample {
    public static void main(String[] args) {
        // Create a transport
        Transport transport = new DefaultTransport("127.0.0.1", 47808);

        // Create a response consumer
        ResponseConsumer<SequenceOf<ObjectIdentifier>> responseConsumer = new ResponseConsumer<>() {
            @Override
            public void handleResponse(SequenceOf<ObjectIdentifier> response) {
                System.out.println("Received response: " + response);
            }

            @Override
            public void handleError(Exception exception) {
                System.err.println("Error handling response: " + exception.getMessage());
            }
        };

        // Send a Who-Is request and register the response consumer
        try {
            transport.send(new WhoIsRequest(), responseConsumer);
        } catch (BACnetException e) {
            e.printStackTrace();
        }

        // Clean up resources
        transport.terminate();
    }
}
```

6. **ResponseConsumerAdapter:**
   The `ResponseConsumerAdapter` class may be an adapter or helper class related to `ResponseConsumer`. It could provide default implementations or convenience methods for handling responses, reducing the boilerplate code required when implementing `ResponseConsumer`.

```java
import com.serotonin.bacnet4j.exception.BACnetException;
import com.serotonin.bacnet4j.service.unconfirmed.WhoIsRequest;
import com.serotonin.bacnet4j.transport.DefaultTransport;
import com.serotonin.bacnet4j.transport.Transport;
import com.serotonin.bacnet4j.type.constructed.SequenceOf;
import com.serotonin.bacnet4j.type.primitive.ObjectIdentifier;

public class BacnetResponseConsumerAdapterExample {
    public static void main(String[] args) {
        // Create a transport
        Transport transport = new DefaultTransport("127.0.0.1", 47808);

        // Create a response consumer adapter
        ResponseConsumerAdapter<SequenceOf<ObjectIdentifier>> responseAdapter = new ResponseConsumerAdapter<>() {
            @Override
            public void handleResponse(SequenceOf<ObjectIdentifier> response) {
                System.out.println("Received response: " + response);
            }
        };

        // Send a Who-Is request and register the response consumer adapter
        try {
            transport.send(new WhoIsRequest(), responseAdapter);
        } catch (BACnetException e) {
            e.printStackTrace();
        }

        // Clean up resources
        transport.terminate();
    }
}
```
7. **ServiceFuture:**
   The `ServiceFuture` class or interface is likely used for managing asynchronous BACnet services. When sending BACnet requests that require asynchronous processing (e.g., read multiple properties, subscribe to notifications), `ServiceFuture` provides a mechanism to track the status of the service request, retrieve results when available, and handle timeouts or errors.
```java
import com.serotonin.bacnet4j.LocalDevice;
import com.serotonin.bacnet4j.RemoteDevice;
import com.serotonin.bacnet4j.exception.BACnetException;
import com.serotonin.bacnet4j.service.unconfirmed.WhoIsRequest;

public class BacnetServiceFutureExample {
    public static void main(String[] args) {
        try {
            // Create a local BACnet device
            LocalDevice localDevice = new LocalDevice(1234);
            localDevice.initialize();

            // Send a Who-Is request to discover other BACnet devices on the network
            ServiceFuture<RemoteDevice> future = localDevice.sendGlobalBroadcast(new WhoIsRequest());

            // Wait for the response and handle it
            RemoteDevice discoveredDevice = future.get();
            System.out.println("Discovered BACnet device: " + discoveredDevice);

            // Clean up resources
            localDevice.terminate();
        } catch (BACnetException | InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
---
## General Use Case:
```java
import com.serotonin.bacnet4j.LocalDevice;
import com.serotonin.bacnet4j.RemoteDevice;
import com.serotonin.bacnet4j.RemoteObject;
import com.serotonin.bacnet4j.exception.BACnetException;
import com.serotonin.bacnet4j.exception.PropertyValueException;
import com.serotonin.bacnet4j.obj.BACnetObject;
import com.serotonin.bacnet4j.service.unconfirmed.WhoIsRequest;
import com.serotonin.bacnet4j.type.constructed.ObjectIdentifier;
import com.serotonin.bacnet4j.type.constructed.PropertyValue;
import com.serotonin.bacnet4j.type.enumerated.PropertyIdentifier;
import com.serotonin.bacnet4j.type.primitive.Real;

public class BacnetCompleteExample {
    public static void main(String[] args) {
        try {
            // Create a local BACnet device with device ID 1234
            LocalDevice localDevice = new LocalDevice(1234);
            localDevice.initialize();

            // Send a Who-Is request to discover other BACnet devices on the network
            ServiceFuture<RemoteDevice> future = localDevice.sendGlobalBroadcast(new WhoIsRequest());

            // Wait for the response and handle it
            RemoteDevice discoveredDevice = future.get();
            System.out.println("Discovered BACnet device: " + discoveredDevice);

            // Get an object (e.g., Analog Value) from the discovered device
            ObjectIdentifier objectId = new ObjectIdentifier(ObjectIdentifier.OBJECT_ANALOG_VALUE, 1);
            RemoteObject remoteObject = discoveredDevice.getObject(objectId);

            // Read a property (e.g., Present Value) from the remote object
            PropertyIdentifier propertyId = PropertyIdentifier.PRESENT_VALUE;
            PropertyValue propertyValue = remoteObject.readProperty(propertyId);
            System.out.println("Value of " + propertyId + ": " + propertyValue);

            // Set a property (e.g., Present Value) on the remote object
            Real newValue = new Real(25.0);
            remoteObject.writeProperty(new PropertyValue(propertyId, newValue));

            // Add a listener to handle notifications for the remote object
            NotificationClassListener notificationListener = new NotificationClassListener() {
                @Override
                public void receivedPropertyChange(RemoteDevice remoteDevice, BACnetObject bacnetObject, PropertyIdentifier propertyIdentifier, PropertyValue propertyValue) {
                    System.out.println("Notification received for property " + propertyIdentifier + " on device " + remoteDevice.getInstanceNumber() + ": " + propertyValue);
                }
            };
            remoteObject.addPropertyChangeListener(notificationListener);

            // Clean up resources
            localDevice.terminate();
        } catch (BACnetException | PropertyValueException | InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
---
## Use Case: HAVOC Controller
Let's create an example where we simulate discovering a remote "HVAC Controller" device and accessing its "Temperature Sensor" object to read the current temperature value. Please note that this is a simplified example for demonstration purposes, and you'll need to adjust the object types, IDs, and property identifiers based on your actual BACnet network setup.

1. **Discovering Remote HVAC Controller Device (BacnetDiscoverHVACControllerExample.java):**
```java
import com.serotonin.bacnet4j.LocalDevice;
import com.serotonin.bacnet4j.RemoteDevice;
import com.serotonin.bacnet4j.exception.BACnetException;
import com.serotonin.bacnet4j.service.unconfirmed.WhoIsRequest;

public class BacnetDiscoverHVACControllerExample {
    public static void main(String[] args) {
        try {
            // Create a local BACnet device with device ID 1234
            LocalDevice localDevice = new LocalDevice(1234);
            localDevice.initialize();

            // Send a Who-Is request to discover other BACnet devices on the network
            localDevice.sendGlobalBroadcast(new WhoIsRequest());

            // Wait for a response and find the remote HVAC controller device
            Thread.sleep(5000); // Wait for 5 seconds to receive responses
            RemoteDevice hvacControllerDevice = null;
            for (RemoteDevice device : localDevice.getRemoteDevices()) {
                if (device.getObjectName().equals("HVAC Controller")) {
                    hvacControllerDevice = device;
                    break;
                }
            }

            if (hvacControllerDevice != null) {
                System.out.println("HVAC Controller Device Found: " + hvacControllerDevice);
            } else {
                System.out.println("HVAC Controller Device Not Found.");
            }

            // Clean up resources
            localDevice.terminate();
        } catch (BACnetException | InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
2. **Accessing Temperature Sensor Object on HVAC Controller (BacnetAccessTemperatureSensorExample.java):**
``` java
import com.serotonin.bacnet4j.LocalDevice;
import com.serotonin.bacnet4j.RemoteDevice;
import com.serotonin.bacnet4j.RemoteObject;
import com.serotonin.bacnet4j.exception.BACnetException;
import com.serotonin.bacnet4j.service.unconfirmed.WhoIsRequest;
import com.serotonin.bacnet4j.type.constructed.ObjectIdentifier;
import com.serotonin.bacnet4j.type.enumerated.ObjectType;
import com.serotonin.bacnet4j.type.enumerated.PropertyIdentifier;
import com.serotonin.bacnet4j.type.primitive.Real;

public class BacnetAccessTemperatureSensorExample {
    public static void main(String[] args) {
        try {
            // Create a local BACnet device with device ID 1234
            LocalDevice localDevice = new LocalDevice(1234);
            localDevice.initialize();

            // Send a Who-Is request to discover other BACnet devices on the network
            localDevice.sendGlobalBroadcast(new WhoIsRequest());

            // Wait for a response and find the remote HVAC controller device
            Thread.sleep(5000); // Wait for 5 seconds to receive responses
            RemoteDevice hvacControllerDevice = null;
            for (RemoteDevice device : localDevice.getRemoteDevices()) {
                if (device.getObjectName().equals("HVAC Controller")) {
                    hvacControllerDevice = device;
                    break;
                }
            }

            if (hvacControllerDevice != null) {
                // Get the temperature sensor object from the HVAC controller device
                ObjectIdentifier objectId = new ObjectIdentifier(ObjectType.analogInput, 1); // Assuming temperature sensor object ID is 1
                RemoteObject temperatureSensorObject = hvacControllerDevice.getObject(objectId);

                // Read the current temperature value from the sensor object
                PropertyIdentifier propertyId = PropertyIdentifier.PRESENT_VALUE;
                Real temperatureValue = (Real) temperatureSensorObject.readProperty(propertyId).getValue();
                System.out.println("Current Temperature: " + temperatureValue);
            } else {
                System.out.println("HVAC Controller Device Not Found.");
            }

            // Clean up resources
            localDevice.terminate();
        } catch (BACnetException | InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
