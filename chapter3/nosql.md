What about NoSQL mechanisms such as Redis, MongoDB, or CouchDB? You unfortunately do not escape from these problems. In order to persist an Aggregate using Redis, you need to serialize it into a string before setting the value. If you use PHP serialize/unserialize methods you will face namespace or class name refactoring issues again. If you choose to serialize in a custom way \(json, custom string, etc.\) you are required to again rebuild the Value Object during Redis retrieval.



