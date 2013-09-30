We introduce a new StructuredClone operator. (Need to inline transferMap semantics from postMessage() to complete this.)

Transferable objects carry a [[Transferable]] internal data property that is either undefined or “neutered”.

Objects defined outside ECMAScript need to define a [[Clone]] internal data property that returns a copy of the object and a new value for the deepClone variable.

The first iteration is not user-pluggable. It is about moving the semantics into ECMAScript proper and tying them down.


= StructuredClone(input, transferMap)

The operator StructuredClone either returns a structured clone of input or throws an exception.

Let memory be an association list of pairs of objects, initially empty.
This is used to handle duplicate references.
In each pair of objects, one is called the source object and the other the destination object.
For each mapping in transferMap, add a mapping from the Transferable object (the source object) 
to the placeholder object (the destination object) to memory.
Return InternalStructuredClone(input, memory)

InternalStructuredClone(input, memory)

The operator StructuredClone either returns a structured clone of input or throws an exception.

If input is the source object of a pair of objects in memory, then return the destination object in that pair of objects.
If input.[[Transferable]] is “neutered”, throw ...
If input is a primitive value, return input.
Let deepClone be false.
If input.[[BooleanData]] exists, return a new Boolean object whose [[BooleanData]] is input.[[BooleanData]].
If input.[[NumberData]] exists, ...
If input.[[StringData]] exists, ...
If input.[[DateValue]] exists, ...
If input.[[RegExpMatcher]] exists, ... also copy [[OriginalSource]] and [[OriginalFlags]].
If input.[[ArrayBufferData]] exists, ...
If input.[[MapData]] exists, ...
If input.[[SetData]] exists, ...
If input.[[ArrayInitialisationState]] exists, ... and set deepClone to true.
Otherwise, if XXX how to check for Object but not Function or Error?
Otherwise, if input.[[Clone]] exists, ...
Otherwise, throw ...
Add a mapping from input (the source object) to output (the destination object) to memory.
If deepClone is true, then, for each enumerable own property in input, run the following steps:
Let name be the name of the property.
Let sourceValue be the result of calling the [[Get]] internal method of input with the argument name. If the [[Get]] internal method of a property involved executing script, and that script threw an uncaught exception, then abort the overall structured clone algorithm, with that exception being passed through to the caller.
Let clonedValue be the result of InternalStructuredClone(sourceValue, memory). If this results in an exception, then abort the overall structured clone algorithm, with that exception being passed through to the caller.
Add a new property to output having the name name, and having the value clonedValue.
Return output.
