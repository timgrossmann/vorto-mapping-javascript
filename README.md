# Mapping arbitrary JSON to Vorto JSON Spec

Vorto allows application developers to create IoT solutions without having to customize their code to deal with the variety of json sent by different kinds of IoT devices. So a temperature reading from a toaster or an aircon will always have the same structure. Vorto achieves this by defining a semantic which abstracts device data using Function Blocks.

Sounds interesting, check out the Eclipse Vorto project [here](https://www.eclipse.org/vorto/).

The package is an effort to making it easier for developers to map arbitrary device json into vorto compliant json structure.

# Installation

    npm install vortomapping



# Example Scenario
I want to create a cool mobile app which shows room temperature. The challenge is there are a wide range of connectable temperature sensors available in the market and my app should be able to work with most of them. A perfect scenario for using Vorto. 

To test my setup I bought a connectable temperature sensor, unfortunately it only reads in Fahrenheit. I am not going to change my application code to adjust to this json format, I will just map it to the desired json format.

Device JSON:

````
{
  "temperature" : "82.4f"
}
````

Desired Vorto compliant JSON structure:

````
{
  "temperature": {
    "status": {
      "sensorValue": "28"
    }
  }
}
````

## Prerequisite
To work through this tutorial, you will need:

- A Github account to log in to Vorto Repository
- A [Vorto Information Model](https://github.com/eclipse/vorto/blob/master/docs/tutorials/describe_tisensor.md), for your IoT device

## Step 1: Create Mapping Specification

Mapping adds device specific information to an Information Model. Since the representation of data can vary from one device to another, we will create a mapping specification to standardize the data.

To create a mapping go to your newly created model and press the **Create Mapping Spec** Button.

![create mapping spec button](./images/create_mapping_key.png)

Now add a Mapping key to uniquely identify your mapping to the device it belongs to, like acme_temperature_monitor (connected temperature sensor from Acme Corporation).

![platform key](./images/specify_mapping_key.png)

Click Create and the web editor opens allowing you to add mapping expression for the Function Blocks you added. You can write XPath 3.1 like notation. Behind the scenes the engine uses [fotonxpath](https://www.npmjs.com/package/fontoxpath) to apply XPath expressions on the DOM created using the device json. To add functionality that may not be possible using xpath alone, you can also use/create custom JavaScript functions (courtesy fotonxpath) . Once you have written your xpath expressions, press Save.

![xpath](./images/xpath.png)

<br></br>
Custom function to convert Fahrenheit to Celsius and round-off to two decimal places.

![xpath](./images/custom_function.png)

## Step 2: Test the Mapping Specification

On the right hand-side, define the device payload(in JSON format) and click **Map**: 

![mapping editor test](./images/mapping_editor_test.png)

Satisfied with the results, save it.

## Step 3: Download & Execute Mapping Specification

Download it to use with our nodejs library:

![download json spec](./images/download_spec.png)


# Usage

    var vortoMappingEngine = require("vorto-mapper");
    
    var rawPayload = {
        "temperature": "82.4f"
    };
    
    var mappingSpec = {}; // copy the downloaded mapping spec
    
    vortoMappingEngine.setMappingSpec(mappingSpec);
    vortoMappingEngine.transform(rawPayload);
    

### Registering Custom Functions

To register user defined functions for the mapping, we will leverage on the functionality provided by [fontoxpath](https://www.npmjs.com/package/fontoxpath). Note the additional parameter 'dynamicContext' which needs to be added as the first parameter of the function.

    vortoMappingEngine.fontoxpath.registerCustomXPathFunction(
        'temperature:fToC',
        ['xs:string'], 'xs:string',
        function fToC(dynamicContext, fahrenheit) 
        {
              var fTemp = fahrenheit.substr(0, fahrenheit.length -1);
              var cel = (fTemp - 32) * 5 / 9;
              return Math.round(cel * 100) /100;
        }
    );

### Debug Logs

The library internally uses [loglevel](https://www.npmjs.com/package/loglevel) npm package. 
To enable debug level logs set the corresponding log level.

    vortoMappingEngine.setLogLevel("debug");