# Structured cloning and transfer
## Overview

Structured cloning algorithm defines the semantics of copying a well-defined subset of ECMAScript 
objects between Code Realms. This algorithm is extensible by host enviroment to support cloning of host objects.

Optionally, some kinds of objects may support a "transfer" operation, the effect of which is to transfer 
"ownership" of some resource associated with an object to a different code realm. 
The object then becomes unusable in source realm. 

----

This specification combines and subsumes http://www.whatwg.org/specs/web-apps/current-work/#dom-messageport-postmessage and 
http://www.whatwg.org/specs/web-apps/current-work/#structured-clone as they really belong together.

HTML spec will refer to this definition for specification of _Structured Clone_ algorithm.

----

We introduce a StructuredClone operator.

Transferable objects carry a [[Transfer]] internal data property that is either a transfer operator or "neutered".

Objects defined outside ECMAScript need to define a [[Clone]] internal data property that returns a copy of the 
object and a new value for the deepClone variable.

Note: _The first iteration is not user-pluggable. It is about moving the semantics into ECMAScript
proper and tying them down._


## StructuredClone(input, transferList, targetRealm)

The operator StructuredClone either returns a _structured clone_ of _input_ or throws an exception.
A _structured clone_ of an object _input_ is an object in code realm _targetRealm_


1. Let memory be a map of source-to-destination object mappings. 

1. For each object transferable in transferList:
    1. If transferable does not have a [[Transfer]] internal data property whose value is an operator, throw ...
    1. Append a mapping from transferable to a new unique placeholder object in memory.
1. Let clone be the result of InternalStructuredClone(input, memory, targetRealm). Re-raise any exceptions.
1. For each object transferable in transferList:
    1. Let transfered be the result of invoking [[Transfer]] on transferable.
    1. Replace the object transferable in memory maps to with transfered.

XXX: We should be clearer about realms here I suppose.

## InternalStructuredClone(input, memory, targetRealm)

The operator InternalStructuredClone either returns a _structured clone_ of _input_ in code realm _targetRealm_
or throws an exception.

1. If input is the source object of a pair of objects in memory, then return the destination object in that pair of objects.
1. If input.[[Transferable]] is “neutered”, throw ...
1. If input is a primitive value, return input.
1. Let deepClone be false.
1. If input.[[BooleanData]] exists, return a new Boolean object whose [[BooleanData]] is input.[[BooleanData]].
1. If input.[[NumberData]] exists, ...
1. If input.[[StringData]] exists, ...
1. If input.[[DateValue]] exists, ...
1. If input.[[RegExpMatcher]] exists, ... also copy [[OriginalSource]] and [[OriginalFlags]].
1. If input.[[ArrayBufferData]] exists, ...
1. If input.[[MapData]] exists, ...
1. If input.[[SetData]] exists, ...
1. If input.[[ArrayInitialisationState]] exists, ... and set deepClone to true.
1. Otherwise, if XXX how to check for Object but not Function or Error?
1. Otherwise, if input.[[Clone]] exists, ...
1. Otherwise, throw ...
1. Add a mapping from input (the source object) to output (the destination object) to memory.
1. If deepClone is true, then, for each enumerable own property in input, run the following steps:
    1. Let name be the name of the property.
    1. Let sourceValue be the result of calling the [[Get]] internal method of input with the argument name. If the [[Get]] internal method of a property involved executing script, and that script threw an uncaught exception, then abort the overall structured clone algorithm, with that exception being passed through to the caller.
1. Let clonedValue be the result of InternalStructuredClone(sourceValue, memory). If this results in an exception, then abort the overall structured clone algorithm, with that exception being passed through to the caller.
1. Add a new property to output having the name name, and having the value clonedValue.
1. Return output.
