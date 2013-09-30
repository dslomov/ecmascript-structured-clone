We introduce a new StructuredClone operator. (Need to inline transferMap semantics from postMessage() to complete this.)

Transferable objects carry a [[Transferable]] internal data property that is either undefined or “neutered”.

Objects defined outside ECMAScript need to define a [[Clone]] internal data property that returns a copy of the object and a new value for the deepClone variable.

The first iteration is not user-pluggable. It is about moving the semantics into ECMAScript proper and tying them down.


# StructuredClone(input, transferMap)

The operator StructuredClone either returns a structured clone of input or throws an exception.

1. Let memory be an association list of pairs of objects, initially empty.
This is used to handle duplicate references.
1. In each pair of objects, one is called the source object and the other the destination object.
For each mapping in transferMap, add a mapping from the Transferable object (the source object) 
to the placeholder object (the destination object) to memory.
1. Return InternalStructuredClone(input, memory)

# InternalStructuredClone(input, memory)

The operator StructuredClone either returns a structured clone of input or throws an exception.

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
