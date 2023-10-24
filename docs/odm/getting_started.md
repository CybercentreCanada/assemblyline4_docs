# Welcome to ODM documentation 📝

??? success "Auto-Generated Documentation"
    This set of documentation is automatically generated from
    [source](https://github.com/CybercentreCanada/assemblyline-base/tree/master/assemblyline/odm)
    and will help ensure any change to functionality will always be documented and available on release.

This section of the site is reserved for advanced use cases where you're interested in how different pieces of data are
stored and structured in Assemblyline.

!!! note "This gives insight into the current functionality of Assemblyline and what you can do."


## Basic Field Types
Here is a table of the basic types of fields in our data models and what they're used for:

|Name|Description|
|:---|:----------|
| `Any` | A field that can hold any value whatsoever but which is stored as a Keyword in the datastore index. |
| `Boolean` | A field storing a boolean value. |
| `Classification` | A field storing access control classification. |
| `ClassificationString` | A field storing the classification as a string only. |
| `Compound` | A field used to wrap Model classes and treat them as fields. |
| `Date` | A field storing a datetime value. |
| `Domain` | A field storing a network domain value. |
| `Email` | A field storing an email address value. |
| `EmptyableKeyword` | A keyword which allow to differentiate between empty and None values. |
| `Enum` | A field storing a short string that has predefined list of possible values. |
| `FlattenedListObject` | A field storing a flattened list object. |
| `FlattenedObject` | A field storing a flattened object. |
| `Float` | A field storing a floating point value. |
| `IP` | A field storing a network IP value. |
| `IndexText` | A special field with special processing rules to simplify searching. |
| `Integer` | A field storing an integer value. |
| `Json` | A field storing serializeable structure with their JSON encoded representations.<br>An example of this is metadata |
| `Keyword` | A field storing a short string with a technical interpretation.<br>Examples: file hashes, service names, document IDs |
| `List` | A field storing a sequence of typed elements. |
| `MAC` | A field storing a MAC address value. |
| `MD5` | A field storing an MD5 hash of a file. |
| `Mapping` | A field storing a sequence of typed elements. |
| `Optional` | A wrapper field that allows simple types (int, float, bool) to take None values. |
| `PhoneNumber` | A field storing a phone number. |
| `Platform` | A field storing an OS platform value. |
| `Processor` | A field storing a processor value. |
| `SHA1` | A field storing an SHA1 hash of a file. |
| `SHA256` | A field storing an SHA256 hash of a file. |
| `SSDeepHash` | A field storing an SSDeepHash value. |
| `Text` | A field storing human-readable text data. |
| `URI` | A field storing a network URI value. |
| `URIPath` | A field storing a network URI path value. |
| `UUID` | A field storing an auto-generated unique ID if None is provided. |
| `UpperKeyword` | A field storing a short uppercase string with a technical interpretation. |
| `ValidatedKeyword` | Keyword field whose value is validated by a regular expression. |


## Field States
In each table, there will be a "Required" column with different states about the field's status:

|State|Description|
|:---|:----------|
|:material-checkbox-marked-outline: Yes|This field is required to be set in the model|
|:material-minus-box-outline: Optional|This field isn't required to be set in the model|
|:material-alert-box-outline: Deprecated|This field has been deprecated in the model. See field's description for more details.|

__Note__: Fields that are ":material-alert-box-outline: Deprecated" that are still shown in the docs will still work as expected but you're encouraged to update your configuration as soon as possible to avoid future deployment issues.
