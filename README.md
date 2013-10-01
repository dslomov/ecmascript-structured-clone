# Structured cloning and transfer
## Overview

Structured cloning algorithm defines the semantics of copying a well-defined subset of ECMAScript 
objects between Code Realms. This algorithm is extensible by host enviroment to support cloning of host objects.

Optionally, some kinds of objects may support a "transfer" operation, the effect of which is to transfer 
"ownership" of some resource associated with an object to a different Code Realm. 
The object then becomes unusable in the source Code Realm. 

----

This specification combines and subsumes http://www.whatwg.org/specs/web-apps/current-work/#dom-messageport-postmessage and 
http://www.whatwg.org/specs/web-apps/current-work/#structured-clone as they really belong together.

HTML spec will be updated to refer to this specification of the _StructuredClone_ algorithm.

----

We introduce a StructuredClone operator.

Transferable objects carry a [[Transfer]] internal data property that is either a transfer operator or "neutered".

Objects defined outside ECMAScript need to define a [[Clone]] internal data property that returns a copy of the 
object and a new value for the deepClone variable.

Note: _The first iteration is not user-pluggable. It is about moving the semantics into ECMAScript
proper and tying them down._


## StructuredClone(input, transferList, targetRealm)

The operator StructuredClone either returns a _structured clone_ of _input_ or throws an exception.
A _structured clone_ of an object _input_ is an object in Code Realm _targetRealm_.

1. Let memory be a map of source-to-destination object mappings. Destination objects are always objects
   of _targetRealm_.
1. For each object transferable in transferList:
    1. If transferable does not have a [[Transfer]] internal data property whose value is an operator, throw ...
    1. Append a mapping from transferable to a new unique placeholder object in memory.
1. Let clone be the result of InternalStructuredClone(input, memory, targetRealm). Re-raise any exceptions.
1. For each object transferable in transferList:
    1. Let transfered be the result of invoking transferable.[[Transfer]]\(targetRealm).
    1. Replace the object transferable in memory maps to with transfered.

## InternalStructuredClone(input, memory, targetRealm)

The operator InternalStructuredClone either returns a _structured clone_ of _input_ in code realm _targetRealm_
or throws an exception.

1. If input is the source object of a pair of objects in memory, then return the destination object in that pair of objects.
1. If input.[[Transferable]] is “neutered”, throw ...
1. If input is a primitive value, return input.
1. Let deepClone be false.
1. If input.[[BooleanData]] exists, 
      return a new Boolean object in _targetRealm_ whose [[BooleanData]] is input.[[BooleanData]].
1. If input.[[NumberData]] exists, 
      return a new Number object in _targetRealm_ whose [[NumberData]] is input.[[NumberData]] 
1. If input.[[StringData]] exists, return a new String object in _targetRealm_ whose [[StringData]] is input.[[StringData]].
1. If input.[[DateValue]] exists, return a new Date object in _targetRealm_ whose [[DateValue]] is input.[[DateValue]].
1. If input.[[RegExpMatcher]] exists, return a new RegExp object _r_ in _targetRealm_ such that: 
    * r.[[RegExpMatcher]] is input.[[RegExpMatcher]]
    * r.[[OriginalSource]] is input.[[OriginalSource]]
    * r.[[OriginalFlags]] is input.[[OriginalFlags]].
1. If input.[[ArrayBufferData]] exists, ...
1. If input.[[MapData]] exists, ...
1. If input.[[SetData]] exists, ...
1. If input is an exotic Array object:
    1. Let _object_ a new Array in _targetRealm_
    2. Set _object_.lenght to _input_.length.
    3. Set _deepClone_ to true
1. Otherwise, if IsCallable( _input_ ), throw ....
1. Otherwise, if input.[[ErrorData]] exists, throw...
1. Otherwise, if input.[[Clone]] exists, ...
1. Otherwise, if input is a host object, throw ...
1. Otherwise: 
    1. Let _object_ be a new Object in _targetRealm_.
    1. set _deepClone_ to true.
2. Add a mapping from _input_ (the source object) to _output_ (the destination object) to _memory_.
3. If _deepClone_ is true:
   1. Let _keys_ be _input_.\[\[OwnPropertyKeys]]\().
   2. For each _key_ in _keys_:
      1. If _key_ is a primitive String value, set _outputKey_ to _key_
      2. TODO: Symbols
      1. Let sourceValue be input.\[\[Get]]\( _key_, _input_)
      2. ReturnIfAbrupt( _sourceValue_ )
      3. Let clonedValue be InternalStructuredClone(sourceValue, memory). 
      4. ReturnIfAbrupt( _clonedValue_ )
      5. output.\[\[Set]]\( _outputKey_, _clonedValue_, _output_)
1. Return _output_.
