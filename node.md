# Salesforce Field Service Node Development Guide

## Tutorial of Declarative Style
Build a declarative-style node#
This tutorial walks through building a declarative-style node. Before you begin, make sure this is the node style you need. Refer to Choose your node building approach for more information.

Existing node

n8n has a built-in NASA node. To avoid clashing with the existing node, you'll give your version a different name.

Step 1: Set up the project#
n8n provides a starter repository for node development. Using the starter ensures you have all necessary dependencies. It also provides a linter.

Clone the repository and navigate into the directory:

Generate a new repository from the template repository.
Clone your new repository:

git clone https://github.com/<your-organization>/<your-repo-name>.git n8n-nodes-nasa-pics
cd n8n-nodes-nasa-pics
The starter contains example nodes and credentials. Delete the following directories and files:

nodes/ExampleNode
nodes/HTTPBin
credentials/ExampleCredentials.credentials.ts
credentials/HttpBinApi.credentials.ts
Now create the following directories and files:

nodes/NasaPics
nodes/NasaPics/NasaPics.node.json
nodes/NasaPics/NasaPics.node.ts
credentials/NasaPicsApi.credentials.ts

These are the key files required for any node. Refer to Node file structure for more information on required files and recommended organization.

Now install the project dependencies:


npm i
Step 2: Add an icon#
Save the NASA SVG logo from here as nasapics.svg in nodes/NasaPics/.

n8n recommends using an SVG for your node icon, but you can also use PNG. If using PNG, the icon resolution should be 60x60px. Node icons should have a square or near-square aspect ratio.

Don't reference Font Awesome

If you want to use a Font Awesome icon in your node, download and embed the image.

Step 3: Create the node#
Every node must have a base file. Refer to Node base file for detailed information about base file parameters.

In this example, the file is NasaPics.node.ts. To keep this tutorial short, you'll place all the node functionality in this one file. When building more complex nodes, you should consider splitting out your functionality into modules. Refer to Node file structure for more information.

Step 5.1: Imports#
Start by adding the import statements:


import { INodeType, INodeTypeDescription } from 'n8n-workflow';
Step 3.2: Create the main class#
The node must export an interface that implements INodeType. This interface must include a description interface, which in turn contains the properties array.

Class names and file names

Make sure the class name and the file name match. For example, given a class NasaPics, the filename must be NasaPics.node.ts.
```

export class NasaPics implements INodeType {
	description: INodeTypeDescription = {
		// Basic node details will go here
		properties: [
		// Resources and operations will go here
		]
	};
}
```
Step 3.3: Add node details#
All nodes need some basic parameters, such as their display name, icon, and the basic information for making a request using the node. Add the following to the description:

```
displayName: 'NASA Pics',
name: 'NasaPics',
icon: 'file:nasapics.svg',
group: ['transform'],
version: 1,
subtitle: '={{$parameter["operation"] + ": " + $parameter["resource"]}}',
description: 'Get data from NASAs API',
defaults: {
	name: 'NASA Pics',
},
inputs: ['main'],
outputs: ['main'],
credentials: [
	{
		name: 'NasaPicsApi',
		required: true,
	},
],
requestDefaults: {
	baseURL: 'https://api.nasa.gov',
	headers: {
		Accept: 'application/json',
		'Content-Type': 'application/json',
	},
},
```
n8n uses some of the properties set in description to render the node in the Editor UI. These properties are displayName, icon, description, and subtitle.

Step 3.4: Add resources#
The resource object defines the API resource that the node uses. In this tutorial, you're creating a node to access two of NASA's API endpoints: planetary/apod and mars-photos. This means you need to define two resource options in NasaPics.node.ts. Update the properties array with the resource object:

```
properties: [
	{
		displayName: 'Resource',
		name: 'resource',
		type: 'options',
		noDataExpression: true,
		options: [
			{
				name: 'Astronomy Picture of the Day',
				value: 'astronomyPictureOfTheDay',
			},
			{
				name: 'Mars Rover Photos',
				value: 'marsRoverPhotos',
			},
		],
		default: 'astronomyPictureOfTheDay',
	},
	// Operations will go here

]
```
type controls which UI element n8n displays for the resource, and tells n8n what type of data to expect from the user. options results in n8n adding a dropdown that allows users to choose one option. Refer to Node UI elements for more information.

Step 3.5: Add operations#
The operations object defines the available operations on a resource.

In a declarative-style node, the operations object includes routing (within the options array). This sets up the details of the API call.

Add the following to the properties array, after the resource object:

```
{
	displayName: 'Operation',
	name: 'operation',
	type: 'options',
	noDataExpression: true,
	displayOptions: {
		show: {
			resource: [
				'astronomyPictureOfTheDay',
			],
		},
	},
	options: [
		{
			name: 'Get',
			value: 'get',
			action: 'Get the APOD',
			description: 'Get the Astronomy Picture of the day',
			routing: {
				request: {
					method: 'GET',
					url: '/planetary/apod',
				},
			},
		},
	],
	default: 'get',
},
{
	displayName: 'Operation',
	name: 'operation',
	type: 'options',
	noDataExpression: true,
	displayOptions: {
		show: {
			resource: [
				'marsRoverPhotos',
			],
		},
	},
	options: [
		{
			name: 'Get',
			value: 'get',
			action: 'Get Mars Rover photos',
			description: 'Get photos from the Mars Rover',
			routing: {
				request: {
					method: 'GET',
				},
			},
		},
	],
	default: 'get',
},
{
	displayName: 'Rover name',
	description: 'Choose which Mars Rover to get a photo from',
	required: true,
	name: 'roverName',
	type: 'options',
	options: [
		{name: 'Curiosity', value: 'curiosity'},
		{name: 'Opportunity', value: 'opportunity'},
		{name: 'Perseverance', value: 'perseverance'},
		{name: 'Spirit', value: 'spirit'},
	],
	routing: {
		request: {
			url: '=/mars-photos/api/v1/rovers/{{$value}}/photos',
		},
	},
	default: 'curiosity',
	displayOptions: {
		show: {
			resource: [
				'marsRoverPhotos',
			],
		},
	},
},
{
	displayName: 'Date',
	description: 'Earth date',
	required: true,
	name: 'marsRoverDate',
	type: 'dateTime',
	default:'',
	displayOptions: {
		show: {
			resource: [
				'marsRoverPhotos',
			],
		},
	},
	routing: {
		request: {
			// You've already set up the URL. qs appends the value of the field as a query string
			qs: {
				earth_date: '={{ new Date($value).toISOString().substr(0,10) }}',
			},
		},
	},
},
```
// Optional/additional fields will go here
This code creates two operations: one to get today's APOD image, and another to send a get request for photos from one of the Mars Rovers. The object named roverName requires the user to choose which Rover they want photos from. The routing object in the Mars Rover operation references this to create the URL for the API call.

Step 3.6: Optional fields#
Most APIs, including the NASA API that you're using in this example, have optional fields you can use to refine your query.

To avoid overwhelming users, n8n displays these under Additional Fields in the UI.

For this tutorial, you'll add one additional field, to allow users to pick a date to use with the APOD endpoint. Add the following to the properties array:

```
{
	displayName: 'Additional Fields',
	name: 'additionalFields',
	type: 'collection',
	default: {},
	placeholder: 'Add Field',
	displayOptions: {
		show: {
			resource: [
				'astronomyPictureOfTheDay',
			],
			operation: [
				'get',
			],
		},
	},
	options: [
		{
			displayName: 'Date',
			name: 'apodDate',
			type: 'dateTime',
			default: '',
			routing: {
				request: {
					// You've already set up the URL. qs appends the value of the field as a query string
					qs: {
						date: '={{ new Date($value).toISOString().substr(0,10) }}',
					},
				},
			},
		},
	],									
}
```
Step 4: Set up authentication#
The NASA API requires users to authenticate with an API key.

Add the following to nasaPicsApi.credentials.ts:

```
import {
	IAuthenticateGeneric,
	ICredentialType,
	INodeProperties,
} from 'n8n-workflow';

export class NasaPicsApi implements ICredentialType {
	name = 'NasaPicsApi';
	displayName = 'NASA Pics API';
	// Uses the link to this tutorial as an example
	// Replace with your own docs links when building your own nodes
	documentationUrl = 'https://docs.n8n.io/integrations/creating-nodes/build/declarative-style-node/';
	properties: INodeProperties[] = [
		{
			displayName: 'API Key',
			name: 'apiKey',
			type: 'string',
			default: '',
		},
	];
	authenticate = {
		type: 'generic',
		properties: {
			qs: {
				'api_key': '={{$credentials.apiKey}}'
			}
		},
	} as IAuthenticateGeneric;
}
```
For more information about credentials files and options, refer to Credentials file.

Step 5: Add node metadata#
Metadata about your node goes in the JSON file at the root of your node. n8n refers to this as the codex file. In this example, the file is NasaPics.node.json.

Add the following code to the JSON file:

```
{
	"node": "n8n-nodes-base.NasaPics",
	"nodeVersion": "1.0",
	"codexVersion": "1.0",
	"categories": [
		"Miscellaneous"
	],
	"resources": {
		"credentialDocumentation": [
			{
				"url": ""
			}
		],
		"primaryDocumentation": [
			{
				"url": ""
			}
		]
	}
}
```
For more information on these parameters, refer to Node codex files.

Step 6: Update the npm package details#
Your npm package details are in the package.json at the root of the project. It's essential to include the n8n object with links to the credentials and base node file. Update this file to include the following information:

```
{
	// All node names must start with "n8n-nodes-"
	"name": "n8n-nodes-nasapics",
	"version": "0.1.0",
	"description": "n8n node to call NASA's APOD and Mars Rover Photo services.",
	"keywords": [
		// This keyword is required for community nodes
		"n8n-community-node-package"
	],
	"license": "MIT",
	"homepage": "https://n8n.io",
	"author": {
		"name": "Test",
		"email": "test@example.com"
	},
	"repository": {
		"type": "git",
		// Change the git remote to your own repository
		// Add the new URL here
		"url": "git+<your-repo-url>"
	},
	"main": "index.js",
	"scripts": {
		// don't change
	},
	"files": [
		"dist"
	],
	// Link the credentials and node
	"n8n": {
		"n8nNodesApiVersion": 1,
		"credentials": [
			"dist/credentials/NasaPicsApi.credentials.js"
		],
		"nodes": [
			"dist/nodes/NasaPics/NasaPics.node.js"
		]
	},
	"devDependencies": {
		// don't change
	},
	"dependencies": {
		// don't change
	}
}
```
You need to update the package.json to include your own information, such as your name and repository URL. For more information on npm package.json files, refer to npm's package.json documentation.

## Code Standards
Following defined code standards when building your node makes your code more readable and maintainable, and helps avoid errors. This document provides guidance on good code practices for node building. It focuses on code details. For UI standards and UX guidance, refer to Node UI design.

Write in TypeScript#
All n8n code is TypeScript. Writing your nodes in TypeScript can speed up development and reduce bugs.

Detailed guidelines for writing a node#
These guidelines apply to any node you build.

Resources and operations#
If your node can perform several operations, call the parameter that sets the operation Operation. If your node can do these operations on more than one resource, create a Resource parameter. The following code sample shows a basic resource and operations setup:

```

export const ExampleNode implements INodeType {
    description: {
        displayName: 'Example Node',
        ...
        properties: [
            {
                displayName: 'Resource',
                name: 'resource',
                type: 'options',
                options: [
                    {
                        name: 'Resource One',
                        value: 'resourceOne'
                    },
                    {
                        name: 'Resource Two',
                        value: 'resourceTwo'
                    }
                ],
                default: 'resourceOne'
            },
            {
                displayName: 'Operation',
                name: 'operation',
                type: 'options',
                // Only show these operations for Resource One
                displayOptions: {
                    show: {
                        resource: [
                            'resourceOne'
                        ]
                    }
                },
                options: [
                    {
                        name: 'Create',
                        value: 'create',
                        description: 'Create an instance of Resource One'
                    }
                ]
            }
        ]
    }
}
```

Reuse internal parameter names#
All resource and operation fields in an n8n node have two settings: a display name, set using the name parameter, and an internal name, set using the value parameter. Reusing the internal name for fields allows n8n to preserve user-entered data if a user switches operations.

For example: you're building a node with a resource named 'Order'. This resource has several operations, including Get, Edit, and Delete. Each of these operations uses an order ID to perform the operation on the specified order. You need to display an ID field for the user. This field has a display label, and an internal name. By using the same internal name (set in value) for the operation ID field on each resource, a user can enter the ID with the Get operation selected, and not lose it if they switch to Edit.

When reusing the internal name, you must ensure that only one field is visible to the user at a time. You can control this using displayOptions.

Detailed guidelines for writing a programmatic-style node#
These guidelines apply when building nodes using the programmatic node-building style. They aren't relevant when using the declarative style. For more information on different node-building styles, refer to Choose your node building approach.

Don't change incoming data#
Never change the incoming data a node receives (data accessible with this.getInputData()) as all nodes share it. If you need to add, change, or delete data, clone the incoming data and return the new data. If you don't do this, sibling nodes that execute after the current one will operate on the altered data and process incorrect data.

It's not necessary to always clone all the data. For example, if a node changes the binary data but not the JSON data, you can create a new item that reuses the reference to the JSON item.

You can see an example in the code of the ReadBinaryFile-Node.

Use the built in request library#
Some third-party services have their own libraries on npm, which make it easier to create an integration. The problem with these packages is that you add another dependency (plus all the dependencies of the dependencies). This adds more and more code, which has to be loaded, can introduce security vulnerabilities, bugs, and so on. Instead, use the built-in module:


// If no auth needed
const response = await this.helpers.httpRequest(options);

// If auth needed
const response = await this.helpers.httpRequestWithAuthentication.call(
	this, 
	'credentialTypeName', // For example: pipedriveApi
	options,
);
This uses the npm package Axios.

Refer to HTTP helpers for more information, and for migration instructions for the deprecated this.helpers.request.

## Project Structure

Node file structure#
Following best practices and standards in your node structure makes your node easier to maintain. It's helpful if other people need to work with the code.

The file and directory structure of your node is affected by:

Your node's complexity.
Whether you use node versioning.
How many nodes you include in the npm package.
Required files and directories#
Your node must include:

A package.json file at the root of the project. This is required for any npm module.
A nodes directory, containing the code for your node:
This directory must contain the base file, in the format <node-name>.node.ts. For example, MyNode.node.ts.
n8n recommends including a codex file, containing metadata for your node. The codex filename must match the node base filename. For example, given a node base file named MyNode.node.ts, the codex name is MyNode.node.json.
The nodes directory can contain other files and subdirectories, including directories for versions, and node code split across more than one file to create a modular structure.
A credentials directory, containing your credentials code. This code lives in a single credentials file. The filename format is <node-name>.credentials.ts. For example, MyNode.credentials.ts.
Modular structure#
You can choose whether to place all your node's functionality in one file, or split it out into a base file and other modules, which the base file then imports. Unless your node is very simple, it's a best practice to split it out.

A basic pattern is to separate out operations. Refer to the HttpBin starter node for an example of this.

For more complex nodes, n8n recommends the following structure:

actions: directory with description and implementation of each possible resource and operation.
In the actions folder, n8n recommends using resources and operations as the names of the sub-folders.
For the implementation and description you can use separate files. Use execute.ts and description.ts as filenames. This makes browsing through the code a lot easier. You can simplify this for nodes that have a less complicated structure.
methods: an optional directory dynamic parameters' functions.
transport: a directory containing the communication implementation.
Versioning#
If your node has more than one version, and you're using full versioning, this makes the file structure more complex. You need a directory for each version, along with a base file that sets the default version. Refer to Node versioning for more information on working with versions, including types of versioning.

Decide how many nodes to include in a package#
There are two possible setups when building a node:

One node in one npm package.
More than one node in a single npm package.
n8n supports both approaches. If you include more than one node, each node should have its own directory in the nodes directory.

A best-practice example for programmatic nodes: The Mattermost node#
n8n's built-in Mattermost node implements a modular structure and versioning, following recommended patterns.

Node base file#
The node base file contains the core code of your node. All nodes must have a base file. The contents of this file are different depending on whether you're building a declarative-style or programmatic-style node. For guidance on which style to use, refer to Choose your node building approach.

This document gives short code snippets to help understand the code structure and concepts. For full walk-throughs of building a node, including real-world code examples, refer to Build a declarative-style node or Build a programmatic-style node.

You can also explore the n8n-nodes-starter and n8n's own nodes for a wider range of examples. The starter contains basic examples that you can build on. The n8n Mattermost node is a good example of a more complex programmatic-style node, including versioning.

Structure of the node base file#
The node base file follows this basic structure:

Import statements
Create a class for the node
Within the node class, create a description object, which defines the node.
A programmatic-style node also has an execute() method, which reads incoming data and parameters, then builds a request. The declarative style handles this using the routing key in the properties object, within descriptions.

Outline structure for a declarative-style node#
This code snippet gives an outline of the node structure.

```
import { INodeType, INodeTypeDescription } from 'n8n-workflow';

export class ExampleNode implements INodeType {
	description: INodeTypeDescription = {
		// Basic node details here
		properties: [
			// Resources and operations here
		]
	};
}

```
Outline structure for a programmatic-style node#
This code snippet gives an outline of the node structure.
```

import { IExecuteFunctions } from 'n8n-core';
import { INodeExecutionData, INodeType, INodeTypeDescription } from 'n8n-workflow';

export class ExampleNode implements INodeType {
	description: INodeTypeDescription = {
    // Basic node details here
    properties: [
      // Resources and operations here
    ]
  };

  async execute(this: IExecuteFunctions): Promise<INodeExecutionData[][]> {
    // Process data and return
  }
};

```
Standard parameters#
These parameters are the same for all node types.

displayName#
String | Required

This is the name users see in the n8n GUI.

name#
String | Required

The internal name of the object. Used to reference it from other places in the node.

icon#
String | Required

Starts with file. For example, icon: 'file:exampleNodeIcon.svg'.

n8n recommends using an SVG for your node icon, but you can also use PNG. If using PNG, the icon resolution should be 60x60px. Node icons should have a square or near-square aspect ratio.

Don't reference Font Awesome

If you want to use a Font Awesome icon in your node, download and embed the image.

group#
Array of strings | Required

Tells n8n how the node behaves when the workflow runs. Options are:

trigger: node waits for a trigger.
schedule: node waits for a timer to expire.
input, output, transform: these currently have no effect.
An empty array, []. Use this as the default option if you don't need trigger or schedule.
description#
String | Required

A short description of the node. n8n uses this in the GUI.

defaults#
Object | Required

Contains essential brand and name settings.

The object can include:

name: String. Used as the node name on the canvas if the displayName is too long.
color: String. Hex color code. Provide the brand color of the integration for use in n8n.
inputs#
Array of strings | Required
```
Names the input connectors. Controls the number of connectors the node has on the input side. If you need only one connector, us input: ['main'].

outputs#
Array of strings | Required

Names the output connectors. Controls the number of connectors the node has on the output side. If you need only one connector, us output: ['main'].
```
credentials#
Array of objects | Required

This parameter tells n8n the credential options. Each object defines an authentication type.

The object must include:

name: the credential name. Must match the name property in the credential file. For example, name: 'asanaApi' in Asana.node.ts links to name = 'asanaApi' in AsanaApi.credential.ts.
required: Boolean. Specify whether authentication is required to use this node.
requestDefaults#
Object | Required

Set up the basic information for the API calls the node makes.

This object must include:

baseURL: The API base URL.
You can also add:

headers: an object describing the API call headers, such as content type.
url: string. Appended to the baseURL. You can usually leave this out. It's more common to provide this in the operations.
properties#
Array of objects | Required

This contains the resource and operations objects that define node behaviors, as well as objects to set up mandatory and optional fields that can receive user input.

Resource objects#
A resource object includes the following parameters:

displayName: String. This should always be Resource.
name: String. This should always be resource.
type: String. Tells n8n which UI element to use, and what type of input to expect. For example, options results in n8n adding a dropdown that allows users to choose one option. Refer to Node UI elements for more information.
noDataExpression: Boolean. Prevents using an expression for the parameter. Must always be true for resource.
Operations objects#
The operations object defines the available operations on a resource.

displayName: String. This should always be Options.
name: String. This should always be option.
type: String. Tells n8n which UI element to use, and what type of input to expect. For example, dateTime results in n8n adding a date picker. Refer to Node UI elements for more information.
noDataExpression: Boolean. Prevents using an expression for the parameter. Must always be true for operation.
options: Array of objects. Each objects describes an operation's behavior, such as its routing, the REST verb it uses, and so on. An options object includes:
name. String.
value. String.
action: String. This parameter combines the resource and operation. You should always include it, as n8n will use it in future versions. For example, given a resource called "Card" and an operation "Get all", your action is "Get all cards".
description: String.
routing: Object containing request details.
Additional fields objects#
These objects define optional parameters. n8n displays them under Additional Fields in the GUI. Users can choose which parameters to set.

The objects must include:
```

displayName: 'Additional Fields',
name: 'additionalFields',
// The UI element type
type: ''
placeholder: 'Add Field',
default: {},
displayOptions: {
  // Set which resources and operations this field is available for
  show: {
    resource: [
      // Resource names
    ],
    operation: [
      // Operation names
    ]
  },
}

```
For more information about UI element types, refer to UI elements.

Declarative-style parameters#
methods and loadOptions#
Object | Optional

methods contains the loadOptions object. You can use loadOptions to query the service to get user-specific settings, then return them and render them in the GUI so the user can include them in subsequent queries. The object must include routing information for how to query the service, and output settings that define how to handle the returned options. For example:

```
methods : {
	loadOptions: {
		routing: {
			request: {
				url: '/webhook/example-option-parameters',
				method: 'GET',
			},
			output: {
				postReceive: [
					{
						// When the returned data is nested under another property
						// Specify that property key
						type: 'rootProperty',
						properties: {
							property: 'responseData',
						},
					},
					{
						type: 'setKeyValue',
						properties: {
							name: '={{$responseItem.key}} ({{$responseItem.value}})',
							value: '={{$responseItem.value}}',
						},
					},
					{
						// If incoming data is an array of objects, sort alphabetically by key
						type: 'sort',
						properties: {
							key: 'name',
						},
					},
				],
			},
		},
	}
},

```
routing#
Object | Required

routing is an object used within an options array in operations and input field objects. It contains the details of an API call.

The code example below comes from the Declarative-style tutorial. It sets up an integration with a NASA API. It shows how to use requestDefaults to set up the basic API call details, and routing to add information for each operation.
```

description: INodeTypeDescription = {
  // Other node info here
  requestDefaults: {
			baseURL: 'https://api.nasa.gov',
			url: '',
			headers: {
				Accept: 'application/json',
				'Content-Type': 'application/json',
			},
		},
    properties: [
      // Resources here
      {
        displayName: 'Operation'
        // Other operation details
        options: [
          {
            name: 'Get'
            value: 'get',
            description: '',
            routing: {
              request: {
                method: 'GET',
                url: '/planetary/apod'
              }
            }
          }
        ]
      }
    ]
}

```


## Credential Files
Credentials file#
The credentials file defines the authorization methods for the node. The settings in this file affect what n8n displays in the Credentials modal, and must reflect the authentication requirements of the service you're connecting to.

In the credentials file, you can use all the n8n UI elements. n8n encrypts the data that's stored using credentials using an encryption key.

Structure of the credentials file
The credentials file follows this basic structure:

```
Import statements
Create a class for the credentials
Within the class, define the properties that control authentication for the node.
Outline structure

import {
	IAuthenticateGeneric,
  ICredentialTestRequest,
	ICredentialType,
	INodeProperties,
} from 'n8n-workflow';

export class ExampleNode implements ICredentialType {
	name = 'exampleNodeApi';
	displayName = 'Example Node API';
	documentationUrl = '';
	properties: INodeProperties[] = [
		{
			displayName: 'API Key',
			name: 'apiKey',
			type: 'string',
			default: '',
		},
	];
	authenticate = {
		type: 'generic',
		properties: {
      // Can be body, header, or qs
			qs: {
        // Use the value from `apiKey` above
				'api_key': '={{$credentials.apiKey}}'
			}

		},
	} as IAuthenticateGeneric;
  test: ICredentialTestRequest = {
    request: {
      baseURL: '={{$credentials?.domain}}',
      url: '/bearer',
    },
  };
}
```
Parameters
name
String. The internal name of the object. Used to reference it from other places in the node.

displayName#
String. The name n8n uses in the GUI.

documentationUrl#
String. URL to your credentials documentation.

properties#
Each object contains:

displayName: the name n8n uses in the GUI.
name: the internal name of the object. Used to reference it from other places in the node.
type: the data type expected, such as string.
default: the URL that n8n should use to test credentials.
authenticate#
Object. Contains objects that tell n8n how to inject the authentication data as part of the API request.

type#
String. If you're using an authentication method that sends data in the header, body, or query string, set this to 'generic'.

properties#
Object. Defines the authentication methods. Options are:

body: object. Sends authentication data in the request body. Can contain nested objects.
header: object. Send authentication data in the request header.
qs: object. Stands for "query string." Send authentication data in the request query string.
test#
Provide a request object containing a URL and authentication type that n8n can use to test the credential.


## The project to build given the above information
Create a new directory named 'node-n8n-salesforce-fieldservice' with the following structure:

```
node-n8n-salesforce-fieldservice/
├── package.json
├── tsconfig.json
├── README.md
├── .gitignore
├── LICENSE
├── GenericFunctions.ts
├── node.json
├── icon.svg
└── nodes/
    ├── SalesforceFieldservice.node.ts
    ├── AssetDescription.ts
    ├── AssetInterface.ts
    ├── AssetDowntimePeriodDescription.ts
    ├── AssetDowntimePeriodInterface.ts
    ├── AssetRelationshipDescription.ts
    ├── AssetRelationshipInterface.ts
    ├── ContractLineItemDescription.ts
    ├── ContractLineItemInterface.ts
    ├── EntitlementDescription.ts
    ├── EntitlementInterface.ts
    ├── EntityMilestoneDescription.ts
    ├── EntityMilestoneInterface.ts
    ├── Pricebook2Description.ts
    ├── Pricebook2Interface.ts
    ├── Product2Description.ts
    ├── Product2Interface.ts
    ├── ServiceContractDescription.ts
    ├── ServiceContractInterface.ts
    ├── SkillDescription.ts
    ├── SkillInterface.ts
    ├── WorkOrderDescription.ts
    ├── WorkOrderInterface.ts
    ├── WorkOrderLineItemDescription.ts
    └── WorkOrderLineItemInterface.ts

```

The most important file is the 'SalesforceFieldservice.node.ts' it should hold all api logic and import each object in.
## API Interaction
Follow Salesforce's best practices for interacting with their API, especially regarding limits. Pass dynamic values and avoid hardcoding as much as possible.

### icon.svg
```
Create an `.svg` file for the salesforce icon to display in the n8n GUI.
```
