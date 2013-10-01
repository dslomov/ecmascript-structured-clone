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


## StructuredClone(_input_, _transferList_, _targetRealm_)

The operator StructuredClone either returns a _structured clone_ of _input_ or throws an exception.
A _structured clone_ of an object _input_ is an object in Code Realm _targetRealm_.

1. Let _memory_ be a map of source-to-destination object mappings.
1. For each object _transferable_ in _transferList_:
    1. If _transferable_ does not have a [[Transfer]] internal data property whose value is an operator, throw ...
    1. Append a mapping from _transferable_ to a new unique placeholder object for _targetRealm_ in _memory_.
1. Let _clone_ be the result of InternalStructuredClone(_input_, _memory_, _targetRealm_).
1. ReturnIfAbrupt(_clone_).
1. For each object _transferable_ in _transferList_:
    1. Let _transfered_ be the result of invoking _transferable_.[[Transfer]]\(_targetRealm_).
    1. Replace the object _transferable_ in _memory_ maps to with _transfered_.

## InternalStructuredClone(_input_, _memory_, _targetRealm_)

The operator InternalStructuredClone either returns a _structured clone_ of _input_ in Code Realm _targetRealm_
or throws an exception.

1. If _input_ is the source object of a pair of objects in _memory_, then return the destination object in that pair of objects.
1. If _input_.[[Transferable]] is “neutered”, throw ...
1. If _input_ is a primitive value, return _input_.
1. Let _deepClone_ be false.
1. If _input_.[[BooleanData]] exists, 
      return a new Boolean object in _targetRealm_ whose [[BooleanData]] is _input_.[[BooleanData]].
1. If _input_.[[NumberData]] exists, 
      return a new Number object in _targetRealm_ whose [[NumberData]] is _input_.[[NumberData]] 
1. If _input_.[[StringData]] exists, return a new String object in _targetRealm_ whose [[StringData]] is _input_.[[StringData]].
1. If _input_.[[DateValue]] exists, return a new Date object in _targetRealm_ whose [[DateValue]] is _input_.[[DateValue]].
1. If _input_.[[RegExpMatcher]] exists, return a new RegExp object _r_ in _targetRealm_ such that: 
    * _r_.[[RegExpMatcher]] is _input_.[[RegExpMatcher]]
    * _r_.[[OriginalSource]] is _input_.[[OriginalSource]]
    * _r_.[[OriginalFlags]] is _input_.[[OriginalFlags]].
1. If _input_.[[ArrayBufferData]] exists, ...
1. If _input_.[[MapData]] exists, ...
1. If _input_.[[SetData]] exists, ...
1. If _input_ is an exotic Array object:
    1. Let _object_ a new Array in _targetRealm_.
    1. Set _object_.lenght to _input_.length.
    1. Set _deepClone_ to true.
1. Otherwise, if IsCallable(_input_), throw ....
1. Otherwise, if input.[[ErrorData]] exists, throw...
1. Otherwise, if input.[[Clone]] exists, ...
1. Otherwise, if input is a host object (what kind of check is this? is this something IDL defines?), throw ...
1. Otherwise: 
    1. Let _object_ be a new Object in _targetRealm_.
    1. set _deepClone_ to true.
1. Add a mapping from _input_ (the source object) to _output_ (the destination object) to _memory_.
1. If _deepClone_ is true:
   1. Let _keys_ be _input_.[[OwnPropertyKeys]]\().
   1. For each _key_ in _keys_:
      1. If _key_ is a primitive String value, set _outputKey_ to _key_
      1. TODO: Symbols
      1. Let _sourceValue_ be _input_.[[Get]]\(_key_, _input_).
      1. ReturnIfAbrupt(_sourceValue_).
      1. Let _clonedValue_ be InternalStructuredClone(_sourceValue_, _memory_). 
      1. ReturnIfAbrupt(_clonedValue_).
      1. output.[[Set]]\( _outputKey_, _clonedValue_, _output_).
1. Return _output_.
