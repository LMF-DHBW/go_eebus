# go_eebus
<img src="https://github.com/LMF-DHBW/go_eebus/blob/master/assets/eebus_logo.png" width="150"> 
This repository includes a framework of the EEBUS protocol in the Go programming language.
The EEBUS protocol suite provides high-quality protocols, which allow efficient communication of smart home IoT devices, independent of device brand, type and other factors.
It provides an open-source, future-proof solution that can be applied in many environments and for numerous applications.

## About
This framework is part of a student research project at DHBW Stuttgart and provides a partial implementation of the EEBUS protocols SHIP and SPINE.

The documentation of the student research project can be found here: 

[GoEEBUS.pdf](https://github.com/LMF-DHBW/go_eebus/blob/main/assets/GoEEBUS.pdf)

The other part of the research project was the implementation of device specific code.
The GitHub repository can be found here: 

[https://github.com/LMF-DHBW/go_eebus_devices](https://github.com/LMF-DHBW/go_eebus_devices)

## Explanation and examples

The following code shows how the framework can be used in general:

````go
// Importing the framework
import eebus "github.com/LMF-DHBW/go-eebus" 

// Configure EEBUS node
eebusNode = eebus.NewEebusNode("100.90.1.102", true, "gateway", "0001", "DHBW", "Gateway")
    
// IP-Adr, is gateway, ssl cert name, device ID, brand name, device type
eebusNode.Update = update // set method called on subscription updates

// Function that creates device structure
buildDeviceModel(eebusNode) 

// Start node
eebusNode.Start() 
````

The following code shows how the device structure for an EEBUS node can be created, in this example an ActuatorSwitch device is created:

````go
eebusNode.DeviceStructure.DeviceType = "Generic"
eebusNode.DeviceStructure.DeviceAddress = "Switch1"
eebusNode.DeviceStructure.Entities = []*resources.EntityModel{
	{
		EntityType:    "Switch",
		EntityAddress: 0,
		Features: []*resources.FeatureModel{
		  // Entitiy 0, Feature 0 always has to be node management
			eebusNode.DeviceStructure.CreateNodeManagement(false),
			{
				FeatureType:    "ActuatorSwitch",
				FeatureAddress: 1,
				Role:           "client",
				Functions:      resources.ActuatorSwitch("button", "button for leds"),
				BindingTo:      []string{"ActuatorSwitch"},
			},
		},
	},
}

// Create node management again, in order to update discovery data
eebusNode.DeviceStructure.Entities[0].Features[0] = eebusNode.DeviceStructure.CreateNodeManagement(false)
````

The gateway device has a list of requests, the following code shows how requests can be accepted from the gateway:

````go
// Requests are saved in the following list:
eebusNode.SpineNode.ShipNode.Requests

i := 0 // Select the first entry as an example

// Accept request by connecting with the device
req := eebusNode.SpineNode.ShipNode.Requests[i]
go eebusNode.SpineNode.ShipNode.Connect(req.Path, req.Ski)

// Remove request from list
eebusNode.SpineNode.ShipNode.Requests = append(eebusNode.SpineNode.ShipNode.Requests[:i], eebusNode.SpineNode.ShipNode.Requests[i+1:]...)
````

How subscription messages can be read:

````go
func update(data resources.DatagramType, conn spine.SpineConnection) {
	entitySource :=     data.Header.AddressSource.Entity
	featureSource :=    data.Header.AddressSource.Feature
	if conn.DiscoveryInformation.FeatureInformation[featureSource].Description.FeatureType == "Measurement" {
		var Function *resources.MeasurementDataType
		err := xml.Unmarshal([]byte(data.Payload.Cmd.Function), &Function)
		// Function.Value contains measured value
	}
}
````

How the states of features can be read:

````go
for _, e := range eebusNode.SpineNode.Connections {
	for _, feature := range e.SubscriptionData {
		if feature.FeatureType == "ActuatorSwitch" && feature.EntityType == "LED" {
			var state *resources.FunctionElement
			err := xml.Unmarshal([]byte(feature.CurrentState), &state)
			// state.Function contains the state (on/off)
		}
	}
}
````

How subscription messages can be send:

````go
for _, e := range eebusNode.SpineNode.Subscriptions {
	e.Send("notify", resources.MakePayload("actuatorSwitchData", 
	resources.FunctionElement{
		Function: "on",
	}))
}
````

How binding messages can be send:

````go
for _, e := range eebusNode.SpineNode.Bindings {
	e.Send("write", resources.MakePayload("actuatorSwitchData", 
	resources.FunctionElement{
		Function: "toggle",
	}))
}
````

## UML diagrams

The framework is devided into the SHIP protocol, the SPINE protocol and defined EEBUS resources.

The following images show the UML diagrams for the framework:

<img src="https://github.com/LMF-DHBW/go-eebus/blob/main/assets/ship_uml.png" width="600"> 

<img src="https://github.com/LMF-DHBW/go-eebus/blob/main/assets/spine_uml.png" width="600">





![Resources Uml part1](https://github.com/LMF-DHBW/go-eebus/blob/main/assets/resources_uml_part1.png)

![Resources Uml part2](https://github.com/LMF-DHBW/go-eebus/blob/main/assets/resources_uml_part2.png)
