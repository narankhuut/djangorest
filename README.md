[TOC]

# API

*(NOTE: All requests require Token Authentication unless specified otherwise.)*

## Nimbus Overview

**Example Scenario**

*SW R&D buys a smart door for it's office.*

Tom Servo is a **User**.

SW R&D is a **Group** of Users. Tom Servo is a member of the SW R&D Group.

"SW R&D's Smart Door" is a **Thing**, as in "Internet-of-*Things*". This Thing is an instance of the "Samsung Smart Door" **Model**.

Tom Servo, SW R&D, and "SW R&D Smart Door" are all **Profiles** in addition to their other types.

"SW R&D's Smart Door" sends **Messages** (ex. `open: true`) to Nimbus, these Messages indicate if the door is open or closed.

Tom Servo sends **Actions** (ex. `set_open`) to "SW R&D's Smart Door", instructing it to open or close.

The "Samsung Smart Door" Model has several **Permissions** that regulate use of the `set_open` Action and permits viewing the `open`  field of Message.

**Permissions** indicate whether a Profile is allowed to send an Action or view a Message to a specific Thing.

A Model has a **Manifest**, which defines how Actions and Messages should look, and what Permissions they require.

~~A `door_control` **Permission** is required to use the `set_open` Action on instances of the "Samsung Smart Door" Model. The "SW R&D's Smart Door" has granted the `door_control` to the "SW R&D" Profile. As Tom Servo is a member of the "SW R&D" Group, he inherits the Permission to send Actions.~~

~~The "Samsung Smart Door" Model has a **Manifest**, the Manifest defines how the `open: true` Message and `set_open: true` Action should be formatted when sent to Nimbus, along with what Permissions the Messages and Actions require.~~



## Requests



Example POST request:

```
POST /users
HTTP/1.1
Host: localhost:8000/api/v1
Authorization: Token 970ed2ec2024c0265287f7abedc60b065b6001f37a94ca6e5d4f66761b0bd5cc
Content-Type: application/json
Body: {
    "email": "user1@email.com",
    "password": "secret_password_123"
}
```



## Authentication

Authentication is required for (nearly) every request. Authentication is token-based, with the exception of the '/auth/login/' which requires only HTTP Basic Authentication (a.k.a username and password). This endpoint is used for distributing the Auth Token (Alternative delivery methods are being discussed). This token expires after a specified limit, currently set to 24 hours, after which it will need to be refreshed.  

### Endpoints

#### `POST /auth/login/`

*(NOTE: This request uses HTTP Basic Authentication)*

The endpoint returns a token specific to the requester and their current device. This token is to be used for all further requests, and needs to be refreshed after 24 hours *(This distribution method is subject to change)*.

*Example request:*

```
POST /token HTTP/1.1
Host: localhost:8000/api/v1
Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
Content-Type: application/json
Body: ""
```

_Example response:_

```python
{
    "token": "970ed2ec2024c0265287f7abedc60b065b6001f37a94ca6e5d4f66761b0bd5cc"
}
```



#### `POST /auth/logout/`

Making a request to this nullifies the posted token.

```
POST /token HTTP/1.1
Host: localhost:8000/api/v1
Authorization: Token 970ed2ec2024c0265287f7abedc60b065b6001f37a94ca6e5d4f66761b0bd5cc
Content-Type: application/json
Body: ""
```



#### `POST /auth/logoutall/`

The endpoint "logoutall" revokes the current token and all other of the requester's profile's token.



## Profiles:

In Nimbus, a Profile refers to an object that users authentication and permissions. Profile Types in Nimbus are User, Group & Thing.

![profiles](images/profiles_basic.png)

![profiles](images/profiles.png)





**Profile Attributes:**

| Attribute | Description                              |
| --------- | ---------------------------------------- |
| `id`      | A randomly generated UUID4 (Note: This id is shared with the Profile's corresponding User/Group/Thing object) |
| `type`    | Specifies the type of Profile; Can be: `user`, `group`, `thing`, or `profile` |



### Endpoints
#### `GET self/`
Get requesting Profile's information.

_Example response:_
```python
{
    "profile_id": "1234",
	"profile_type": "user",
  	...
}
```

## User:
User is a model that exists to represent a singular person.

**User Attributes:**

| Attribute | Description                 |
| --------- | --------------------------- |
| `id`      | A randomly generated UUID4. |
| `email`   | User's email address.       |



### User Endpoints:

#### `GET users/`
Returns a list of users requester has access to.

_Example response:_
```python
{
    "count": 2,
    "next": "http://localhost:8000/api/v1/users/?limit=2&offset=2",
    "previous": null,
    "results": [
        {
            "id": "31947d6d-8f55-4f2e-b395-e9ba799da15b",
            "created": 1508224209297,
            "modified": 1508224209328,
            "email": "user1@email.com",
        },
        {
            "id": "355d6197-21c2-4c62-9b27-b85c047f701c",
            "created": 1508224209296,
            "modified": 1508224209296,
            "email": "user2@email.com",
        }
    ]
}
```



#### `POST users/`

Create a user. (Note: this is temporary, and will eventually be replaced/improved.)

_Example request body:_
```python
{
    "email": "user1@email.com",
    "password": "secret_password_123"
}
```



#### `GET users/[user_id]/`

Get a specific User's information. (assuming you have access to this user)

_Example response:_
```python
{
    "id": "355d6197-21c2-4c62-9b27-b85c047f701c",
    "created": 1508224209296,
    "modified": 1508224209296,
    "email": "user1@email.com",
}
```



#### `PUT users/[user_id]/`

Replace ALL of an existing user's information.

_Example request body:_
```python
{
    "email": "new_user1@email.com",
    "password": "new_secret_password_123"
}
```



#### `PATCH users/[user_id]/`

Replace defined portions of an existing user's information.

_Example request body:_
```python
{
    "email": "new_user1@email.com",
}
```



#### `DELETE users/[user_id]/`

Delete a User the requester has access to.



### Nested Endpoints:

#### `GET users/[user_id]/things/`
Get a list of all of a user's things.

_Example reponse:_
```python
{
    "count": 2,
    "next": null,
    "previous": null,
    "results": [
        {
            "id": "bb4d97a8-32fe-4c3e-95cc-21820ad692c6",
            "name": "Thing",
            "model": "a1dbf40daa9b41559409df295792051f",
            "linked_profiles": [
                "ccbb845f-5f16-45bb-8b29-9c3260d8c332"
            ],
            "created": 1508224209782,
            "modified": 1508224209782
        },
        {
            "id": "1b59da58-deb6-42b5-87cf-59ab00edf171",
            "name": "Thing",
            "model": "a1dbf40daa9b41559409df295792051f",
            "linked_profiles": [
                "12db8cf8-3481-4aa1-b1d4-b12433e82c96"
            ],
            "created": 1508224209847,
            "modified": 1508224209847
        }
    ]
}
```

#### `GET users/[user_id]/things/[thing_id]`

Get a specific thing attached to a user.

_Example Response:_
```python
{
    "id": "bb4d97a8-32fe-4c3e-95cc-21820ad692c6",
    "name": "Thing",
    "model": "a1dbf40daa9b41559409df295792051f",
    "linked_profiles": [
        "ccbb845f-5f16-45bb-8b29-9c3260d8c332"
    ],
    "created": 1508224209782,
    "modified": 1508224209782
}
```

#### `GET users/[user_id]/messages/`
Get a list of all of a specific User's accessible messages.

*Example Response:*

```python
{
    "count": 10,
    "next": null,
    "previous": null,
    "results": [
        {
            "id": "fc7d6757-fcb7-4f51-a8ae-cb97e63d8903",
            "thing": "bb4d97a832fe4c3e95cc21820ad692c6",
            "data": {
                "admin_view": true,
                "staff_view": true,
                "staff_and_admin_view": true
            },
            "created": 1508224209799,
            "modified": 1508224209799
        },
	...
	]
}
```

#### `GET users/[user_id]/actions/`

Get a list of all actions sent by a specific User. ~~(Maybe this should be changed to Profile?)~~

*Example Response*

```python
{
    "count": 2,
    "next": null,
    "previous": null,
    "results": [
        {
            "id": "3f0e992c-d987-46f6-a6d9-1c7001f445aa",
            "created": 1508322618954,
            "modified": 1508322618954,
            "data": {
                "lock": "False"
            },
            "name": "set_lock",
            "thing": "df00e1cb335149099ab20cf5097267bc",
            "sender": "ccbb845f5f1645bb8b299c3260d8c332"
        },
        ...
    ]
}
```



## Groups

![profiles](images/groups.png)

A Group is a group of Profiles. As a Group is also a Profile, it can be granted Permissions to Things. In this manner you can have a Group (e.g. "Employees") that all have access to an object ("Office Printer"), but should any Profile leave or join the Group, the Permissions sort themselves out automatically. This leads to simple, effective access control.

| Attribute | Description                              |
| --------- | ---------------------------------------- |
| `id`      | Randomly generated UUID4                 |
| `name`    | Name of Group                            |
| `members` | List of Profiles that belong to the Group. |

### Group Endpoints

#### `GET groups/`

Returns a list of groups that the requester is a part of.

*Example Response:*

```python
{
    "count": 2,
    "next": null,
    "previous": null,
    "results": [
        {
            "id": "12db8cf8-3481-4aa1-b1d4-b12433e82c96",
            "created": 1508224209375,
            "modified": 1508224209390,
            "name": "Office Employees",
            "members": [
                "31947d6d-8f55-4f2e-b395-e9ba799da15b",
                "355d6197-21c2-4c62-9b27-b85c047f701c",
                "6fdb0e4a-36c6-4c37-b17c-57066f6531ac",
                "729f6018-afcb-4159-b68f-920012d628c4",
                "ccbb845f-5f16-45bb-8b29-9c3260d8c332",
                "ce91ce89-cfb9-4884-9d88-1f8873806e41"
            ]
        },
      	...
    ]
}
```

#### `POST groups/`

Creates a new Group.

(May change to a Group 'invitation' in the future.)

*Example request body:*

```python
{
  "name": "Family Group",
  "members": [
    "31947d6d-8f55-4f2e-b395-e9ba799da15b", # Profile ID's
    "355d6197-21c2-4c62-9b27-b85c047f701c",
    "729f6018-afcb-4159-b68f-920012d628c4"
  ]
}
```



#### `GET groups/[group_id]/`

Get a specific Group.

*Example Response:*

```python
{
    "id": "12db8cf8-3481-4aa1-b1d4-b12433e82c96",
    "created": 1508224209375,
    "modified": 1508224209390,
  	"name": "Team Canada"
    "members": [
        "31947d6d-8f55-4f2e-b395-e9ba799da15b",
        "355d6197-21c2-4c62-9b27-b85c047f701c",
        "6fdb0e4a-36c6-4c37-b17c-57066f6531ac"
    ]
}
```



#### `PUT groups/[group_id]/`
Modify an entire pre-existing Group.

*Example Request Body:*

```python
{
  	"name": "Team Canada v2"
    "members": [
      "729f6018-afcb-4159-b68f-920012d628c4",
      "ccbb845f-5f16-45bb-8b29-9c3260d8c332",
      "ce91ce89-cfb9-4884-9d88-1f8873806e41"
    ]
}
```



#### `PATCH groups/[group_id]/`

Modify a specific section of Group, leaving everything else the same.

*Example Request Body:*

```python
{
    "name": "Team that needed a new name"
}
```



#### `DELETE groups/[group_id]/`

Deletes specified Group. ~~(Will eventually limit this to Group Admin.)~~



### Nested Group Endpoints

#### `GET groups/[group_id]/things/`

~~(I probably need to add groups/[group_id]/things/[thing_id]/permissions)~~

Returns a list of Things that have been assigned to the Group.

*Example Response:*

```python
{
    "count": 3,
    "next": null,
    "previous": null,
    "results": [
        {
            "id": "1b59da58-deb6-42b5-87cf-59ab00edf171",
            "name": "Thing",
            "model": "a1dbf40daa9b41559409df295792051f",
            "linked_profiles": [
                "12db8cf8-3481-4aa1-b1d4-b12433e82c96"
            ],
            "created": 1508224209847,
            "modified": 1508224209847
        },
      	...
    ]
}
```



#### `GET groups/[group_id]/profiles/`

~~(Maybe under profiles I want to add POST/PUT etc. ?)~~

Returns list of Profiles attached to Group.



## Things

![profiles](images/things.png)

The Thing model represents the "Things" in Internet of Things. A Thing can be a device, an application, or anything else that represents an internet connected object.

A Thing sends messages, which represent it's current status/ 'properties'. Other Profiles can send Actions for a Thing to perform.

Both Actions and Messages are defined by a Thing's Model.

Permissions are set somewhere else.

| Attribute         | Description                              |
| ----------------- | ---------------------------------------- |
| `id`              | Randomly Generated UUID4                 |
| `name`            | The Thing's colloquial name.             |
| `model`           | The Model used to define this Thing's characteristics. |
| `linked_profiles` | A list of profiles that are allowed any level of interaction with this Thing. |
| `created`         | Unix Time Stamp (in ms) denoting when Thing was created. |
| `modified`        | Unix Time Stamp (in ms) denoting when Thing was last modified. |



### Thing Endpoints

#### `GET things/`

Get list of Things available to all of requester's Profiles.



#### `POST things/`

Create new Thing.

```python
{
    "count": 3,
    "next": null,
    "previous": null,
    "results": [
        {
            "id": "bb4d97a8-32fe-4c3e-95cc-21820ad692c6",
            "name": "Thing",
            "model": "a1dbf40daa9b41559409df295792051f",
            "linked_profiles": [
                "ccbb845f-5f16-45bb-8b29-9c3260d8c332"
            ],
            "created": 1508224209782,
            "modified": 1508224209782
        },
      	...
    ]
}
```



#### `GET things/[thing_id]/`

Get specific Thing.

```python
{
    "id": "1b59da58-deb6-42b5-87cf-59ab00edf171",
    "name": "Thing",
    "model": "a1dbf40daa9b41559409df295792051f",
    "linked_profiles": [
        "12db8cf8-3481-4aa1-b1d4-b12433e82c96"
    ],
    "created": 1508224209847,
    "modified": 1508224209847
}
```



#### `PUT things/[thing_id]/`

Replace entire Thing.

```python
{
    "name": "Replacing Old Thing",
    "model": "a1dbf40daa9b41559409df295792051f",
    "linked_profiles": [
        "12db8cf8-3481-4aa1-b1d4-b12433e82c96"
    ]
}
```

#### `PATCH things/[thing_id]/`

Replace specific Thing attributes.

```python
{
    "linked_profiles": [
        "12db8cf8-3481-4aa1-b1d4-b12433e82c96",
        "ccbb845f-5f16-45bb-8b29-9c3260d8c332"
    ],
}
```

#### `DELETE things/[thing_id]/`

Delete specified Thing.



### Nested Thing Endpoints

#### `GET things/[thing_id]/profiles/`

~~Perhaps doesn't need to exist~~

Get list of Profiles linked to Thing.

```python
{
  "count": 2,
  "next": "http://localhost:8000/api/v1/users/?limit=2&offset=2",
  "previous": null,
  "results": [
      {
          "id": "31947d6d-8f55-4f2e-b395-e9ba799da15b",
          "created": 1508224209297,
          "modified": 1508224209328,
          "email": "user1@email.com",
      },
      ...
  ]
}
```


#### `GET things/[thing_id]/messages/`
Get list of Messages sent by Thing

```python
{
    "count": 5,
    "next": null,
    "previous": null,
    "results": [
        {
            "id": "0356d54e-8745-441f-a545-72101a7d1202",
            "thing": "1b59da58deb642b587cf59ab00edf171",
            "data": {
                "admin_view": true,
                "staff_view": true,
                "staff_and_admin_view": true
            },
            "created": 1508224209878,
            "modified": 1508224209878
        },
      	...
    ]
}
```



#### `GET things/[thing_id]/actions/`

Get list of Actions sent to Thing.

```python
{
    "count": 2,
    "next": null,
    "previous": null,
    "results": [
        {
            "id": "3f0e992c-d987-46f6-a6d9-1c7001f445aa",
            "created": 1508322618954,
            "modified": 1508322618954,
            "data": {
                "lock": "False"
            },
            "name": "set_lock",
            "thing": "df00e1cb335149099ab20cf5097267bc",
            "sender": "ccbb845f5f1645bb8b299c3260d8c332"
        },
        ...
    ]
}
```



#### `GET things/[thing_id]/permissions/`

Get list of Thing's Permissions.

```python
{
    "count": 2,
    "next": null,
    "previous": null,
    "results": [
        {
            "id": "e0d3136f-ba2c-4fba-8d45-a19b13670255",
            "name": "admin",
            "profile": [
                "12db8cf8-3481-4aa1-b1d4-b12433e82c96"
            ]
        },
	    ...
    ]
}
```



## Models

![profiles](images/models.png)

Models work to define how Things behave. Models define how Actions and Messages need to be formatted in order to successfully be sent to/by a Thing.

*Example*: The Model of a Samsung Smart Door defines how

If a Thing is a singular instance of a Samsung Smart Door, the Model is a software representation of how  

| Attribute     | Description                              |
| ------------- | ---------------------------------------- |
| `id`          | Randomly Generated UUID4                 |
| `unique_name` | A name required to be unique that specifies what this model is. (i.e. com.samsung.smartdoor) |
| `name`        | A more recognizable name. (i.e. "Samsung Smart Door") |
| `manifests`   | A list of Manifest's belonging to the Model. |
| `created`     | Unix Time Stamp (in ms) denoting when Model was created. |
| `modified`    | Unix Time Stamp (in ms) denoting when Model was last modified. |

### Model Endpoints

#### `GET models/`

Get list of Models.

```python
{
    "count": 2,
    "next": null,
    "previous": null,
    "results": [
        {
            "id": "674f4705-e1bb-41ce-b596-71d28e1aa1bf",
            "created": 1508322601966,
            "modified": 1508322601966,
            "unique_name": "Model 3b6f4016-ec4a-4f43-8971-6c3f8afd8c48",
            "name": "IoT Door Model"
        },
        ...
    ]
}
```



#### `POST models/`

Create a new Model.

```python
{
	"name": "IoT Door",
  	"unique_name" "com.thiscompany.iotdoorv1"
}
```



#### `GET models/[model_id]/`

Get specific Model.

```python
{
    "id": "674f4705-e1bb-41ce-b596-71d28e1aa1bf",
    "created": 1508322601966,
    "modified": 1508322601966,
    "unique_name": "Model 3b6f4016-ec4a-4f43-8971-6c3f8afd8c48",
    "name": "IoT Door Model"
}
```



#### `PUT models/[model_id]/`

Replace entire Model.

```python
{
    "unique_name": "com.thiscompany.iotdoorv2",
    "name": "IoT Door Model v2"
}
```



#### `PATCH models/[model_id]/`

Replace sections of specific Model, leaving the rest intact.

```python
{
     "name": "It's Another IoT Door Model"
}
```



#### `DELETE models/[model_id]/`

Delete specified Model.



### Nested Model Endpoints
#### `GET models/[model_id]/manifests/`

Get a list of specified Model's Manifests.

```python
{
    "count": 1,
    "next": null,
    "previous": null,
    "results": [
        {
            "id": "0393f9ad-68f6-4276-b22f-b722fb2b34ae",
            "schema": {
                "actions": [
                    {
                        "data": [
                            {
                                "name": "open",
                                "datatype": "boolean"
                            }
                        ],
                        "action": "set_open",
                        "permissions": [
                            "is_admin"
                        ]
                    },
                ],
                "properties": [
                    {
                        "name": "open",
                        "datatype": "boolean",
                        "permissions": [
                            "is_staff",
                            "is_admin"
                        ]
                    }
                ],
                "permissions": {
                    "permission_list": [
                        "is_staff",
                        "is_admin"
                    ],
                    "creator_profile_default": [
                        "is_staff",
                        "is_admin"
                    ],
                    "connected_profile_default": [
                        "is_staff"
                    ]
                }
            },
            "model": "674f4705e1bb41ceb59671d28e1aa1bf"
        }
    ]
}
```



## Manifests

![profiles](images/manifests.png)

### Manifest Endpoints

The Manifest is a schema used to validate incoming Messages and Actions for specific Models. Additionally, a Manifest dictates the Permissions required to view Message properties and allow specific Actions.

Look to creating a manifest at here: LINK.

The default Manifest for a given Model is the newest Manifest, but the endpoint supports using earlier Manifest versions.

| Attribute  | Description                              |
| ---------- | ---------------------------------------- |
| `id`       | Randomly generated UUID4                 |
| `model`    | The Model this Manifest is built for.    |
| `schema`   | The JSON schema used to define incoming Actions, Messages, and their Permissions. See here[LINK] for creating a Manifest schema. |
| `version`  | Indicates the Manifest version, set automatically. |
| `created`  | Unix Time Stamp (in ms) denoting when Manifest was created. |
| `modified` | Unix Time Stamp (in ms) denoting when Manifest was last modified. |

#### `GET manifests/`

~~(Possibly getting removed)~~

Get list of all Manifests.

```python
{
    "count": 2,
    "next": null,
    "previous": null,
    "results": [
        {
            "id": "928bc4f9-acaa-459c-8239-cb2264407215",
          	"model": "674f4705-e1bb-41ce-b596-71d28e1aa1bf",
            "version": 1,
            "schema": {
				[...]
            },
            "created": 1508322601966,
            "modified": 1508322601966
        }
    ]
}
```



#### `POST manifests/`

Create new Manifest.  See creating Manifest Schema [LINK]

```python
{
    "schema":{
        [...]
    },
  	"model": "928bc4f9-acaa-459c-8239-cb2264507215"
}
```



#### `GET manifests/[manifest_id]/`

Get specific Manifest.

```python
{
    "id": "928bc4f9-acaa-459c-8239-cb2264407215",
    "model": "674f4705-e1bb-41ce-b596-71d28e1aa1bf",
    "version": 1,
    "schema": {
      [...]
    },
    "created": 1508322601966,
    "modified": 1508322601966
}
```



#### `PUT manifests/[manifest_id]/`

Replace Manifest.

```python
{
    "model": "674f4705-e1bb-41ce-b596-71d28e1aa1bf",
    "schema": {
      [...]
    },
}
```



#### `PATCH manifests/[manifest_id]/`

Replace sections of Manifest.

```python
{
    "schema": {
      [...]
    },
}
```



#### `DELETE manifests/[manifest_id]/`

Delete specified Manifest.



## Messages

![profiles](images/messages.png)

A Message is a small JSON object sent by a Thing to share its information.

| Attribute  | Description                              |
| ---------- | ---------------------------------------- |
| `id`       | Randomly generated UUID4                 |
| `thing`    | Reference to Thing that is sending the Message. |
| `data`     | The payload of information, a JSON object defined by the Message's Thing's Model's Manifest. |
| `created`  | Unix Time Stamp (in ms) denoting when Message was created. |
| `modified` | Unix Time Stamp (in ms) denoting when Manifest was last modified. |



### Message Endpoints

#### `GET messages/`

Get list of messages sent by Things that requester has access to.

```python
{
    "count": 10,
    "next": null,
    "previous": null,
    "results": [
        {
            "id": "fc7d6757-fcb7-4f51-a8ae-cb97e63d8903",
            "thing": "bb4d97a832fe4c3e95cc21820ad692c6",
            "data": {
                "open": true,
                "lock": false,
            },
            "created": 1508224209799,
            "modified": 1508224209799
        },
      	...
    ]
}
```

#### `POST messages/`

Create a new Message.

```python
{
    "thing": "fc7d6757-fcb7-4f51-a8ae-cb97e63d8903",
    "data": {
      "open": true,
      "lock": false,
    },
}
```



#### `GET messages/[message_id]/`

Get a specific Message (that a requester has access to.)

```python
{
    "id": "fc7d6757-fcb7-4f51-a8ae-cb97e63d8903",
    "thing": "bb4d97a832fe4c3e95cc21820ad692c6",
    "data": {
      "open": true,
      "lock": false,
    },
    "created": 1508224209799,
    "modified": 1508224209799
}
```

#### `PUT messages/[message_id]/`

Replace entirety of specified Message.

```python
{
    "thing": "fc7d6757-fcb7-4f51-a8ae-cb97e63d8903",
    "data": {
      "open": false,
      "lock": true,
    }
}
```



#### `PATCH messages/[message_id]/`

Replace specified portion of Message, leaving the other sections intact.

```python
{
    "data": {
      "open": false,
      "lock": false,
    }
}
```



#### `DELETE messages/[message_id]/`

Delete specific Message.



## Actions

![profiles](images/actions.png)

Actions are a JSON object sent by Profiles to a Thing. `action` names the Action that will happen, `data` includes relevant data. (e.g. "`{"action": "set_door_status", "data": {"open": "true", "lock":"false"}`")

The Thing uses these Action objects as an indication that it should perform a task.  

| Attribute  | Description                              |
| ---------- | ---------------------------------------- |
| `id`       | Randomly generated UUID4                 |
| `thing`    | Thing action is being sent to.           |
| `action`   | Name of action that will be performed by Thing |
| `data`     | The payload of information, a JSON object defined by the Action's Thing's Model's Manifest. |
| `sender`   | The Profile that is sending the Action.  |
| `created`  | Unix Time Stamp (in ms) denoting when Action was created. |
| `modified` | Unix Time Stamp (in ms) denoting when Action was last modified. |



### Action Endpoints

#### `GET actions/`

Get list of all actions sent by requester.

```python
{
    "count": 4,
    "next": null,
    "previous": null,
    "results": [
        {
            "id": "3f0e992c-d987-46f6-a6d9-1c7001f445aa",
            "created": 1508322618954,
            "modified": 1508322618954,
            "data": {
                "lock": "False"
            },
            "action": "set_lock",
            "thing": "df00e1cb335149099ab20cf5097267bc",
            "sender": "ccbb845f5f1645bb8b299c3260d8c332"
        },
      	...
    ]
}
```



#### `POST actions/`

Send a new Action.

```python
{
    "id": "3f0e992c-d987-46f6-a6d9-1c7001f445aa",
    "created": 1508322618954,
    "modified": 1508322618954,
    "data": {
      "lock": "False"
    },
    "action": "set_lock",
    "thing": "df00e1cb335149099ab20cf5097267bc",
    "sender": "ccbb845f5f1645bb8b299c3260d8c332"
}
```



#### `GET actions/[action_id]/`

Get a specific Action.

```python
{
    "id": "3f0e992c-d987-46f6-a6d9-1c7001f445aa",
    "created": 1508322618954,
    "modified": 1508322618954,
    "data": {
        "lock": "False"
    },
    "action": "set_lock",
    "thing": "df00e1cb335149099ab20cf5097267bc",
    "sender": "ccbb845f5f1645bb8b299c3260d8c332"
}
```



#### `PUT actions/[action_id]/`

Replace an entire Action.

```python
{
    "data": {
        "lock": "True"
    },
    "action": "set_lock",
    "thing": "df00e1cb335149099ab20cf5097267bc",
    "sender": "ccbb845f5f1645bb8b299c3260d8c332"
}
```

#### `PATCH actions/[action_id]/`

Replace specified fields of specified Action, leaving the rest intact.

```python
{
    "data": {
        "lock": "False"
    }
}
```

#### `DELETE actions/[action_id]/`

Delete specified Action.



## Permissions

![profiles](images/permissions.png)

A Permission belongs to a Thing, and connects to any number of Profiles. Permissions are automatically created when a Manifest is uploaded.

When sending an Action, Nimbus finds the appropriate Permissions and checks if the sender is in the `linked_profiles`.  When viewing Messages, if the requester does not have the appropriate Permission to view a field, that field is removed. `

| Attribute         | Description                              |
| ----------------- | ---------------------------------------- |
| `id`              |                                          |
| `name`            | The name of the Permission. (This name is fuctional.) |
| `thing`           | Thing that requires this Permission.     |
| `linked_profiles` | List of all profile that have this Permission |
| `created`         | Unix Time Stamp (in ms) denoting when Permission was created. |
| `modified`        | Unix Time Stamp (in ms) denoting when Permission was last modified. |



### Permission Endpoints

#### `GET permissions/`

Get list of all Permissions requesting Profile has.

```python
{
    "count": 16,
    "next": "http://localhost:8000/api/v1/permissions/?limit=10&offset=10",
    "previous": null,
    "results": [
        {
            "id": "20bb17cf-dedc-48fa-9cbd-9ff06563ae0d",
            "name": "staff",
          	"thing": "ccbb845f5f1645bb8b299c3260d8c332"
            "profile": [
                "355d6197-21c2-4c62-9b27-b85c047f701c"
            ],
          	"created": 1508322618954,
          	"modified": 1508322618954
        },
        ...
    ]
}
```

#### `POST permissions/`

~~We might end up disabling this~~

Create a new Permission.

```python
{
    "name": "staff",
    "thing": "ccbb845f5f1645bb8b299c3260d8c332"
    "profile": [
      "355d6197-21c2-4c62-9b27-b85c047f701c"
    ]
}
```



#### `GET permissions/[permission_id]/`

Get specific Permission.

```python
{
    "id": "20bb17cf-dedc-48fa-9cbd-9ff06563ae0d",
    "name": "staff",
    "thing": "ccbb845f5f1645bb8b299c3260d8c332"
    "profile": [
      "355d6197-21c2-4c62-9b27-b85c047f701c"
    ],
    "created": 1508322618954,
    "modified": 1508322618954
}
```



#### `PUT permissions/[permission_id]/`

Replace specified Permission in its entirety.

```python
{
    "name": "staff",
    "thing": "ccbb845f5f1645bb8b299c3260d8c332"
    "profile": [
      "355d6197-21c2-4c62-9b27-b85c047f701c",
      "40cc17cf-dedc-48fa-9cbd-9ff06563ae0d"
    ]
}
```



#### `PATCH permissions/[permission_id]/`

Modify specified attributes of Permission, leaving other attributes intact.

```python
{
    "profile": [
      "40cc17cf-dedc-48fa-9cbd-9ff06563ae0d"
    ]
}
```



#### `DELETE permissions/[permission_id]/`

Delete specified Permission.



## Endpoint List

```python
# profiles
GET self/

# users
GET users/
POST users/
GET users/[user_id]/
PUT users/[user_id]/
PATCH users/[user_id]/
DELETE users/[user_id]/

# users nested
GET users/[user_id]/things/
GET users/[user_id]/things/[thing_id]
GET users/[user_id]/messages/
GET users/[user_id]/actions/

# groups
GET groups/
POST groups/
GET groups/[group_id]/
PUT groups/[group_id]/
PATCH groups/[group_id]/
DELETE groups/[group_id]/

# groups nested
GET groups/[group_id]/things/
GET groups/[group_id]/users/

# things
GET things/
POST things/
GET things/[thing_id]/
PUT things/[thing_id]/
PATCH things/[thing_id]/
DELETE things/[thing_id]/

# things nested
GET things/[thing_id]/profiles/
GET things/[thing_id]/messages/
GET things/[thing_id]/actions/
GET things/[thing_id]/permissions/

# models
GET models/
POST models/
GET models/[model_id]/
PUT models/[model_id]/
PATCH models/[model_id]/
DELETE models/[model_id]/

# models nested
GET models/[model_id]/manifests/

# manifests
GET manifests/
POST manifests/
GET manifests/[manifest_id]/
PUT manifests/[manifest_id]/
PATCH manifests/[manifest_id]/
DELETE manifests/[manifest_id]/

# messages
GET messages/
POST messages/
GET messages/[message_id]/
PUT messages/[message_id]/
PATCH messages/[message_id]/
DELETE messages/[message_id]/

# actions
GET actions/
POST actions/
GET actions/[action_id]/
PUT actions/[action_id]/
PATCH actions/[action_id]/
DELETE actions/[action_id]/

# permissions
GET permissions/
POST permissions/
GET permissions/[permission_id]/
PUT permissions/[permission_id]/
PATCH permissions/[permission_id]/
DELETE permissions/[permission_id]/
```
