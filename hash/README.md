## Hash table with Quadratic Probing

You will create a `HashTable` data structure that uses open-addressing (i.e. uses probing for collision resolution) and **NOT** chaining.  Your table should support **linear** and **quadratic** probing via functors.  Initially, we will use the `std::hash<>` functor provided with the C++ `std` library.  In a future question, you will implement your own hash function for **string**s made of letters and digits only (no punctuation). You should complete the implementation in `ht.h`.

```c++
template<
    typename K, 
    typename V, 
    typename Prober = LinearProber,
    typename Hash = std::hash<K>, 
    typename KEqual = std::equal_to<K> >
class HashTable
{
    /* Implementation */
};
```

The template types are as follows:
  - **K**: the key type (just like a normal map)
  - **V**: the value type (just like a normal map)
  - **Prober**: a functor that supports an `init()` and `next()` function that the hash table can use to generate the probing sequence (e.g. linear, quadratic, double-hashing).  A base `Prober` class has been written and a derived `LinearProber` is nearly complete. You will then write your own `QuadraticProber` class to implement quadratic probing.
  - **Hash**:  The hash function that converts a value of type **K** to a `std::size_t` which we have typedef'd as `HASH_INDEX_T`.
  - **KEqual**: A functor that takes two **K** type objects and should return `true` if the two objects are **equal** and `false`, otherwise.

Internally, we will use a hash table of **pointers** to `HashItem`s, which have the following members:

```c++
typedef std::pair<KeyType, ValueType> ItemType;
struct HashItem {
        ItemType item;  // key, value pair
        bool deleted;
};
```

You should allocate a `HashItem` when inserting and free them at an appropriate time based on the description below.  If no location is available to insert the item, you should throw `std::logic_error`.

We have also provided a constant array of prime sizes that can be used, in turn, when resizing and rehashing is necessary.  This sequence is: 11, 23, 47, 97, 197, 397, 797, 1597, 3203, 6421, 12853, 25717, 51437, 102877, 205759, 411527, 823117, 1646237, 3292489, 6584983, 13169977, 26339969, 52679969, 105359969, 210719881, 421439783, 842879579, 1685759167.  Thus, when a HashTable is constructed it should start with a **size of 11**. The client will supply a **loading factor**, `alpha`, at construction, and before inserting any new element, you should resize (to the next larger size in the list above) if the current loading factor is **at or above** the given `alpha`.  

For removal of keys, we will use a deferred strategy of simply marking an item as "deleted" using a `bool` value.  These values should not be considered when calls are made to `find` or `operator[]` but should continue to count toward the loading factor until the next resize/rehash.  At that point, when you resize, you will only rehash the non-deleted items and (permanently) remove the deleted items.

We will not ask you to implement an `iterator` for the hash table.  So `find()` will simply return a pointer to the key,value pair if it exists and `nullptr` if it does not.

To facilitate tracking relevant statistics we will use for performance measurements, we have provided the core **probing** routine: `probe(key)`. This routine, applies the hash function to get a starting index, then initializes the prober and repeatedly calls next() until it finds the desired key, reaches a hashtable location that contains `nullptr` (where a new item may be inserted if desired), or reaches its upper limit of calls (i.e. cannot find a null location).  `probe()` utilizes the `Prober`.  `Prober::init` is called to give the prober all relevant info that it may need, regardless of the probing strategy (i.e. starting index, table size, etc.).  Then a sequence of calls to `Prober::next()` will ensue.  If, for example we are using linear probing, the first call to `next()` would yield h(k), the subsequent call to `next()` would yield h(k)+1, the third call would yield h(k)+2, etc.  Notice: the first call to `next()` just returns the h(k) value passed to `Prober::init()`.  As probing progresses we will update statistics such as the total number of probes.   Some accessor and mutator functions are provided to access those statistics.

Note:  When probing to insert a new key, we could stop on a location marked as deleted and simply overwrite it with the key to be inserted.  However, because our design uses the same `probe(key)` function for all three operations: insert, find, and remove, and certainly for find and remove we would NOT want to stop on a deleted item, but instead keep probing to find the desired key, we will simply take the more inefficient probing approach of not reusing locations marked for deletion when probing for insertion.

A debug function, `reportAll` is also provided to print out each key value pair. Use this when necessary to help you debug your code.

#### Probers

We have abstracted the probing sequence so various strategies may be implemented without the hash table implementation being modified.  Complete the `LinearProber` and then write a similar `QuadraticProber` that uses quadratic probing. Recall that if a prime table size is used, insertion using quadratic probing is only guaranteed to find a free location if the loading factor is less than or equal to **m/2**. Thus after m/2 probes, return `npos`.

#### Testing Your Hash Table

A simple test driver program `ht-test.cpp` has been provided which you may modify to test various aspects of your hash table implementation. We will not grade this file so use it as you please.  We will run our own suite of tests to determine credit for this assignment.