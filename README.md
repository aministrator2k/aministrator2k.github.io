
# Blog

## Things I wish I knew before working with Kafka & Avro in .net
A while back I started looking into using Schema Registry for serializing and deserliazing kafka messages. This enables automatic serialization and descerialization of messages to and from strongly typed CLR objects when working with Confluent's Kafka .net consumer/publisher library. Below is a few things that stalled me for a bit that I wish I knew beforehand:

### 1. Every nested record must be nullable
My first schema worked perfectly when publishing messages but as soon as my consumer was to descerlize the message I ran into these weird null reference errors; the dreaded "Object reference not set to instance of object". This baffled me for the longest time as my schema looked completely fine:
```json
{
    "type": "record",
    "name": "Entity",
    "namespace": "My.Namespace.Model",
    "fields": [
        {
            "name": "Id",
            "type": [
                "string"
            ]
        },
        {
            "name": "NestedEntity",
            "type": [
                {
                    "type": "record",
                    "name": "NestedType",
                    "fields": [
                        {
                            "name": "Id",
                            "type": [
                                "string"
                            ]
                        }
                    ]
                }
            ]
        }
    ]
}
```
This  I realized because the Avro Serdes library at some point tries to access the nested object which is not initialized yet. Thankfully the solution was as easy as making the nested objects nullable. Notice how the array of types for the nested object now includes null as a valid type:
```json
{
    "type": "record",
    "name": "Entity",
    "namespace": "My.Namespace.Model",
    "fields": [
        {
            "name": "Id",
            "type": [
                "string"
            ]
        },
        {
            "name": "NestedEntity",
            "type": [
                "null",
                {
                    "type": "record",
                    "name": "NestedType",
                    "fields": [
                        {
                            "name": "Id",
                            "type": [
                                "string"
                            ]
                        }
                    ]
                }
            ]
        }
    ]
}
```
### 2. There is an implicit dependency on field in your model classes
The way I usually approach a new technology is to try to re-make the hello world application from scratch based on official examples. This process almost works as a step by step guide easing you into the new ecosystem , dependencies and their relations. Here again I tried to follow the examples in confluent's Github but quickly ran into weird error messages about schema not being defined. I kept scratching my head comparing my code with examples until I realized the only difference is that model classes in the examples have a public static property named ``_Schema`` but this couldn't be it right? After looking at the Avro Serdes code turned out that was indeed the problem! Your classes must expose this variable which should be initialized with your schema string.

### 3. Do not use arrays
The Avro Serdes library by Confluent does not like arrays! This I found the hard way after wasting a lot of time trying to figure out why my arrays are not being de-serialized. After changing all my arrays to lists everything worked as expected!
