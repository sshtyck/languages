Format String Syntax
********************

The ``str.format()`` method and the ``Formatter`` class share the same
syntax for format strings (although in the case of ``Formatter``,
subclasses can define their own format string syntax).

Format strings contain "replacement fields" surrounded by curly braces
``{}``. Anything that is not contained in braces is considered literal
text, which is copied unchanged to the output.  If you need to include
a brace character in the literal text, it can be escaped by doubling:
``{{`` and ``}}``.

The grammar for a replacement field is as follows:

      replacement_field ::= "{" [field_name] ["!" conversion] [":" format_spec] "}"
      field_name        ::= arg_name ("." attribute_name | "[" element_index "]")*
      arg_name          ::= [identifier | integer]
      attribute_name    ::= identifier
      element_index     ::= integer | index_string
      index_string      ::= <any source character except "]"> +
      conversion        ::= "r" | "s"
      format_spec       ::= <described in the next section>

In less formal terms, the replacement field can start with a
*field_name* that specifies the object whose value is to be formatted
and inserted into the output instead of the replacement field. The
*field_name* is optionally followed by a  *conversion* field, which is
preceded by an exclamation point ``'!'``, and a *format_spec*, which
is preceded by a colon ``':'``.  These specify a non-default format
for the replacement value.

See also the *Format Specification Mini-Language* section.

The *field_name* itself begins with an *arg_name* that is either a
number or a keyword.  If it's a number, it refers to a positional
argument, and if it's a keyword, it refers to a named keyword
argument.  If the numerical arg_names in a format string are 0, 1, 2,
... in sequence, they can all be omitted (not just some) and the
numbers 0, 1, 2, ... will be automatically inserted in that order.
Because *arg_name* is not quote-delimited, it is not possible to
specify arbitrary dictionary keys (e.g., the strings ``'10'`` or
``':-]'``) within a format string. The *arg_name* can be followed by
any number of index or attribute expressions. An expression of the
form ``'.name'`` selects the named attribute using ``getattr()``,
while an expression of the form ``'[index]'`` does an index lookup
using ``__getitem__()``.

Changed in version 2.7: The positional argument specifiers can be
omitted, so ``'{} {}'`` is equivalent to ``'{0} {1}'``.

Some simple format string examples:

   "First, thou shalt count to {0}" # References first positional argument
   "Bring me a {}"                  # Implicitly references the first positional argument
   "From {} to {}"                  # Same as "From {0} to {1}"
   "My quest is {name}"             # References keyword argument 'name'
   "Weight in tons {0.weight}"      # 'weight' attribute of first positional arg
   "Units destroyed: {players[0]}"  # First element of keyword argument 'players'.

The *conversion* field causes a type coercion before formatting.
Normally, the job of formatting a value is done by the
``__format__()`` method of the value itself.  However, in some cases
it is desirable to force a type to be formatted as a string,
overriding its own definition of formatting.  By converting the value
to a string before calling ``__format__()``, the normal formatting
logic is bypassed.

Two conversion flags are currently supported: ``'!s'`` which calls
``str()`` on the value, and ``'!r'`` which calls ``repr()``.

Some examples:

   "Harold's a clever {0!s}"        # Calls str() on the argument first
   "Bring out the holy {name!r}"    # Calls repr() on the argument first

The *format_spec* field contains a specification of how the value
should be presented, including such details as field width, alignment,
padding, decimal precision and so on.  Each value type can define its
own "formatting mini-language" or interpretation of the *format_spec*.

Most built-in types support a common formatting mini-language, which
is described in the next section.

A *format_spec* field can also include nested replacement fields
within it. These nested replacement fields can contain only a field
name; conversion flags and format specifications are not allowed.  The
replacement fields within the format_spec are substituted before the
*format_spec* string is interpreted. This allows the formatting of a
value to be dynamically specified.

See the *Format examples* section for some examples.


Format Specification Mini-Language
==================================

"Format specifications" are used within replacement fields contained
within a format string to define how individual values are presented
(see *Format String Syntax*).  They can also be passed directly to the
built-in ``format()`` function.  Each formattable type may define how
the format specification is to be interpreted.

Most built-in types implement the following options for format
specifications, although some of the formatting options are only
supported by the numeric types.

A general convention is that an empty format string (``""``) produces
the same result as if you had called ``str()`` on the value. A non-
empty format string typically modifies the result.

The general form of a *standard format specifier* is:

   format_spec ::= [[fill]align][sign][#][0][width][,][.precision][type]
   fill        ::= <a character other than '}'>
   align       ::= "<" | ">" | "=" | "^"
   sign        ::= "+" | "-" | " "
   width       ::= integer
   precision   ::= integer
   type        ::= "b" | "c" | "d" | "e" | "E" | "f" | "F" | "g" | "G" | "n" | "o" | "s" | "x" | "X" | "%"

The *fill* character can be any character other than '{' or '}'.  The
presence of a fill character is signaled by the character following
it, which must be one of the alignment options.  If the second
character of *format_spec* is not a valid alignment option, then it is
assumed that both the fill character and the alignment option are
absent.

The meaning of the various alignment options is as follows:

   +-----------+------------------------------------------------------------+
   | Option    | Meaning                                                    |
   +===========+============================================================+
   | ``'<'``   | Forces the field to be left-aligned within the available   |
   |           | space (this is the default for most objects).              |
   +-----------+------------------------------------------------------------+
   | ``'>'``   | Forces the field to be right-aligned within the available  |
   |           | space (this is the default for numbers).                   |
   +-----------+------------------------------------------------------------+
   | ``'='``   | Forces the padding to be placed after the sign (if any)    |
   |           | but before the digits.  This is used for printing fields   |
   |           | in the form '+000000120'. This alignment option is only    |
   |           | valid for numeric types.                                   |
   +-----------+------------------------------------------------------------+
   | ``'^'``   | Forces the field to be centered within the available       |
   |           | space.                                                     |
   +-----------+------------------------------------------------------------+

Note that unless a minimum field width is defined, the field width
will always be the same size as the data to fill it, so that the
alignment option has no meaning in this case.

The *sign* option is only valid for number types, and can be one of
the following:

   +-----------+------------------------------------------------------------+
   | Option    | Meaning                                                    |
   +===========+============================================================+
   | ``'+'``   | indicates that a sign should be used for both positive as  |
   |           | well as negative numbers.                                  |
   +-----------+------------------------------------------------------------+
   | ``'-'``   | indicates that a sign should be used only for negative     |
   |           | numbers (this is the default behavior).                    |
   +-----------+------------------------------------------------------------+
   | space     | indicates that a leading space should be used on positive  |
   |           | numbers, and a minus sign on negative numbers.             |
   +-----------+------------------------------------------------------------+

The ``'#'`` option is only valid for integers, and only for binary,
octal, or hexadecimal output.  If present, it specifies that the
output will be prefixed by ``'0b'``, ``'0o'``, or ``'0x'``,
respectively.

The ``','`` option signals the use of a comma for a thousands
separator. For a locale aware separator, use the ``'n'`` integer
presentation type instead.

Changed in version 2.7: Added the ``','`` option (see also **PEP
378**).

*width* is a decimal integer defining the minimum field width.  If not
specified, then the field width will be determined by the content.

If the *width* field is preceded by a zero (``'0'``) character, this
enables zero-padding.  This is equivalent to an *alignment* type of
``'='`` and a *fill* character of ``'0'``.

The *precision* is a decimal number indicating how many digits should
be displayed after the decimal point for a floating point value
formatted with ``'f'`` and ``'F'``, or before and after the decimal
point for a floating point value formatted with ``'g'`` or ``'G'``.
For non-number types the field indicates the maximum field size - in
other words, how many characters will be used from the field content.
The *precision* is not allowed for integer values.

Finally, the *type* determines how the data should be presented.

The available string presentation types are:

   +-----------+------------------------------------------------------------+
   | Type      | Meaning                                                    |
   +===========+============================================================+
   | ``'s'``   | String format. This is the default type for strings and    |
   |           | may be omitted.                                            |
   +-----------+------------------------------------------------------------+
   | None      | The same as ``'s'``.                                       |
   +-----------+------------------------------------------------------------+

The available integer presentation types are:

   +-----------+------------------------------------------------------------+
   | Type      | Meaning                                                    |
   +===========+============================================================+
   | ``'b'``   | Binary format. Outputs the number in base 2.               |
   +-----------+------------------------------------------------------------+
   | ``'c'``   | Character. Converts the integer to the corresponding       |
   |           | unicode character before printing.                         |
   +-----------+------------------------------------------------------------+
   | ``'d'``   | Decimal Integer. Outputs the number in base 10.            |
   +-----------+------------------------------------------------------------+
   | ``'o'``   | Octal format. Outputs the number in base 8.                |
   +-----------+------------------------------------------------------------+
   | ``'x'``   | Hex format. Outputs the number in base 16, using lower-    |
   |           | case letters for the digits above 9.                       |
   +-----------+------------------------------------------------------------+
   | ``'X'``   | Hex format. Outputs the number in base 16, using upper-    |
   |           | case letters for the digits above 9.                       |
   +-----------+------------------------------------------------------------+
   | ``'n'``   | Number. This is the same as ``'d'``, except that it uses   |
   |           | the current locale setting to insert the appropriate       |
   |           | number separator characters.                               |
   +-----------+------------------------------------------------------------+
   | None      | The same as ``'d'``.                                       |
   +-----------+------------------------------------------------------------+

In addition to the above presentation types, integers can be formatted
with the floating point presentation types listed below (except
``'n'`` and None). When doing so, ``float()`` is used to convert the
integer to a floating point number before formatting.

The available presentation types for floating point and decimal values
are:

   +-----------+------------------------------------------------------------+
   | Type      | Meaning                                                    |
   +===========+============================================================+
   | ``'e'``   | Exponent notation. Prints the number in scientific         |
   |           | notation using the letter 'e' to indicate the exponent.    |
   +-----------+------------------------------------------------------------+
   | ``'E'``   | Exponent notation. Same as ``'e'`` except it uses an upper |
   |           | case 'E' as the separator character.                       |
   +-----------+------------------------------------------------------------+
   | ``'f'``   | Fixed point. Displays the number as a fixed-point number.  |
   +-----------+------------------------------------------------------------+
   | ``'F'``   | Fixed point. Same as ``'f'``.                              |
   +-----------+------------------------------------------------------------+
   | ``'g'``   | General format.  For a given precision ``p >= 1``, this    |
   |           | rounds the number to ``p`` significant digits and then     |
   |           | formats the result in either fixed-point format or in      |
   |           | scientific notation, depending on its magnitude.  The      |
   |           | precise rules are as follows: suppose that the result      |
   |           | formatted with presentation type ``'e'`` and precision     |
   |           | ``p-1`` would have exponent ``exp``.  Then if ``-4 <= exp  |
   |           | < p``, the number is formatted with presentation type      |
   |           | ``'f'`` and precision ``p-1-exp``. Otherwise, the number   |
   |           | is formatted with presentation type ``'e'`` and precision  |
   |           | ``p-1``. In both cases insignificant trailing zeros are    |
   |           | removed from the significand, and the decimal point is     |
   |           | also removed if there are no remaining digits following    |
   |           | it.  Positive and negative infinity, positive and negative |
   |           | zero, and nans, are formatted as ``inf``, ``-inf``, ``0``, |
   |           | ``-0`` and ``nan`` respectively, regardless of the         |
   |           | precision.  A precision of ``0`` is treated as equivalent  |
   |           | to a precision of ``1``.                                   |
   +-----------+------------------------------------------------------------+
   | ``'G'``   | General format. Same as ``'g'`` except switches to ``'E'`` |
   |           | if the number gets too large. The representations of       |
   |           | infinity and NaN are uppercased, too.                      |
   +-----------+------------------------------------------------------------+
   | ``'n'``   | Number. This is the same as ``'g'``, except that it uses   |
   |           | the current locale setting to insert the appropriate       |
   |           | number separator characters.                               |
   +-----------+------------------------------------------------------------+
   | ``'%'``   | Percentage. Multiplies the number by 100 and displays in   |
   |           | fixed (``'f'``) format, followed by a percent sign.        |
   +-----------+------------------------------------------------------------+
   | None      | The same as ``'g'``.                                       |
   +-----------+------------------------------------------------------------+


Format examples
===============

This section contains examples of the new format syntax and comparison
with the old ``%``-formatting.

In most of the cases the syntax is similar to the old
``%``-formatting, with the addition of the ``{}`` and with ``:`` used
instead of ``%``. For example, ``'%03.2f'`` can be translated to
``'{:03.2f}'``.

The new format syntax also supports new and different options, shown
in the follow examples.

Accessing arguments by position:

   >>> '{0}, {1}, {2}'.format('a', 'b', 'c')
   'a, b, c'
   >>> '{}, {}, {}'.format('a', 'b', 'c')  # 2.7+ only
   'a, b, c'
   >>> '{2}, {1}, {0}'.format('a', 'b', 'c')
   'c, b, a'
   >>> '{2}, {1}, {0}'.format(*'abc')      # unpacking argument sequence
   'c, b, a'
   >>> '{0}{1}{0}'.format('abra', 'cad')   # arguments' indices can be repeated
   'abracadabra'

Accessing arguments by name:

   >>> 'Coordinates: {latitude}, {longitude}'.format(latitude='37.24N', longitude='-115.81W')
   'Coordinates: 37.24N, -115.81W'
   >>> coord = {'latitude': '37.24N', 'longitude': '-115.81W'}
   >>> 'Coordinates: {latitude}, {longitude}'.format(**coord)
   'Coordinates: 37.24N, -115.81W'

Accessing arguments' attributes:

   >>> c = 3-5j
   >>> ('The complex number {0} is formed from the real part {0.real} '
   ...  'and the imaginary part {0.imag}.').format(c)
   'The complex number (3-5j) is formed from the real part 3.0 and the imaginary part -5.0.'
   >>> class Point(object):
   ...     def __init__(self, x, y):
   ...         self.x, self.y = x, y
   ...     def __str__(self):
   ...         return 'Point({self.x}, {self.y})'.format(self=self)
   ...
   >>> str(Point(4, 2))
   'Point(4, 2)'

Accessing arguments' items:

   >>> coord = (3, 5)
   >>> 'X: {0[0]};  Y: {0[1]}'.format(coord)
   'X: 3;  Y: 5'

Replacing ``%s`` and ``%r``:

   >>> "repr() shows quotes: {!r}; str() doesn't: {!s}".format('test1', 'test2')
   "repr() shows quotes: 'test1'; str() doesn't: test2"

Aligning the text and specifying a width:

   >>> '{:<30}'.format('left aligned')
   'left aligned                  '
   >>> '{:>30}'.format('right aligned')
   '                 right aligned'
   >>> '{:^30}'.format('centered')
   '           centered           '
   >>> '{:*^30}'.format('centered')  # use '*' as a fill char
   '***********centered***********'

Replacing ``%+f``, ``%-f``, and ``% f`` and specifying a sign:

   >>> '{:+f}; {:+f}'.format(3.14, -3.14)  # show it always
   '+3.140000; -3.140000'
   >>> '{: f}; {: f}'.format(3.14, -3.14)  # show a space for positive numbers
   ' 3.140000; -3.140000'
   >>> '{:-f}; {:-f}'.format(3.14, -3.14)  # show only the minus -- same as '{:f}; {:f}'
   '3.140000; -3.140000'

Replacing ``%x`` and ``%o`` and converting the value to different
bases:

   >>> # format also supports binary numbers
   >>> "int: {0:d};  hex: {0:x};  oct: {0:o};  bin: {0:b}".format(42)
   'int: 42;  hex: 2a;  oct: 52;  bin: 101010'
   >>> # with 0x, 0o, or 0b as prefix:
   >>> "int: {0:d};  hex: {0:#x};  oct: {0:#o};  bin: {0:#b}".format(42)
   'int: 42;  hex: 0x2a;  oct: 0o52;  bin: 0b101010'

Using the comma as a thousands separator:

   >>> '{:,}'.format(1234567890)
   '1,234,567,890'

Expressing a percentage:

   >>> points = 19.5
   >>> total = 22
   >>> 'Correct answers: {:.2%}'.format(points/total)
   'Correct answers: 88.64%'

Using type-specific formatting:

   >>> import datetime
   >>> d = datetime.datetime(2010, 7, 4, 12, 15, 58)
   >>> '{:%Y-%m-%d %H:%M:%S}'.format(d)
   '2010-07-04 12:15:58'

Nesting arguments and more complex examples:

   >>> for align, text in zip('<^>', ['left', 'center', 'right']):
   ...     '{0:{fill}{align}16}'.format(text, fill=align, align=align)
   ...
   'left<<<<<<<<<<<<'
   '^^^^^center^^^^^'
   '>>>>>>>>>>>right'
   >>>
   >>> octets = [192, 168, 0, 1]
   >>> '{:02X}{:02X}{:02X}{:02X}'.format(*octets)
   'C0A80001'
   >>> int(_, 16)
   3232235521
   >>>
   >>> width = 5
   >>> for num in range(5,12):
   ...     for base in 'dXob':
   ...         print '{0:{width}{base}}'.format(num, base=base, width=width),
   ...     print
   ...
       5     5     5   101
       6     6     6   110
       7     7     7   111
       8     8    10  1000
       9     9    11  1001
      10     A    12  1010
      11     B    13  1011

Related help topics: OPERATORS

