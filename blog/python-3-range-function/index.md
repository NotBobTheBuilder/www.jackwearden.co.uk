Something I noticed after today's [Brum Codejo](http://brumcodejo.github.io/) was a new feature of Python's range function.

The [`range` method in Python 2](http://docs.python.org/2/library/functions.html#range) accepts an optional start index, an end index and an optional step count and returns a list, like so:

(Note that the last element is always less than any supplied end index. This means `len(range(5)) == 5`)

    range(5)        == [0, 1, 2, 3, 4]
    range(1, 5)     == [1, 2, 3, 4]
    range(1, 10, 2) == [1, 3, 5, 7, 9]

There's a change in Python 3 - `range` is now a type. This means that `range(2, 3)` is essentially a constructor.

Instances of `range` are still iterable, so `for i in range(n)` loops will still work fine, but the values are computed lazily, as with [generators](http://www.python.org/dev/peps/pep-0255/). This means that range can make some comparisons cleaner.

For example, in [Conway's Game of Life](https://en.wikipedia.org/wiki/Conways_Game_of_Life), (which we attempted at today's CodeJo), cells live or die depending on the number of neighbors each cell has. This can make use of Python's chained comparisons and be implemented as:

    cell_lives = 3 >= neighbour_count >= 2

using the new range, this could also be rewritten as follows:

    cell_lives = neighbour_count in range(2, 4)

Admittedly, that doesn't grant much of an improvement; however it could be useful when you also need to check modulo. For example, to check if a number is a multiple of three between 60 and 90:

    (90 > number >= 60) and (number % 3 == 0)

could be replaced by

    number in range(60, 90, 3)

While this is still possible in Python 2, it requires the construction of the whole list, which is then searched. In Python 3, this list isn't built, the `__contains__` method on the range object determines if the number lies in the sequence.

Also note that this isn't the same as Python 2's [`xrange`](http://docs.python.org/2/library/functions.html#xrange), which only supports lazy generation and indexing - with `xrange` a call to `__contains__` with the `in` keyword performs a linear search for a value.

Besides that, I'm convinced this is useful, but I can't think of a use of it.
