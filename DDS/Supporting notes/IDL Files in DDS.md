An IDL (Interface Definition Language) file is used to define data types and interfaces in a language agnostic way. 

The complete DDS IDL specification may be found [here](obsidian://open?vault=Obsidian&file=DDS%2FSupporting%20Documents%2FIDLSpecification.pdf), but its quite verbose.

In the context of DDS, IDL files are used to define the topic and the message which is passed. Examine the IDL file from the messenger example in the JAVA OpenDDS implementation:

```
module Messenger {

  @topic
  struct Message {
    string from;
    string subject;
    @key long subject_id;
    string text;
    unsigned long count;
  };
};
```

The module `Messenger` defines the namespace - similar to namespaces in C++ or packages in JAVA.

The `@topic` indicates that that the following struct (`Message`) represents a DDS topic, which means instances of this struct can be published and subscribed to in a DDS system.

The struct `Message` defines the message, with the following fields:
- `string from`: A string representing the sender of the message.
- `string subject`: A string representing the subject of the message.
- `@key long subject_id`: A long integer uniquely identifying the message. This is marked as a key field.
- `string text`: A string representing the body of the message.
- `unsigned long count`: An unsigned long integer representing a count related to the message (e.g., number of times it has been sent).

