# cJSON accept an invalid text

### 1.Introduction

This is a bug related to Expected Behavior Violation, cJSON incorrectly accepts unescaped control characters in json string.

Specifically, as documented in Section 7 of the [specification][1]（i.e., "... the characters that must be escaped: ... the control characters (U+0000 through U+001F)".

```
A string begins and ends with quotation marks.  
All Unicode characters may be placed within the quotation marks, 
except for the characters that must be escaped:
quotation mark, reverse solidus, and the control characters (U+0000 through U+001F).
```

json parser is expected to reject the text if it contains unescaped control characters.

However, we found a case that cJSON fails to reject it.
Other JSON parsers, such as those found at [github.com/nlohmann/json][3] and [github.com/simdjson/simdjson][2], were able to parse the text without issue.

*potential effect*: This may lead to some unexpected errors, e.g., data out-of-sync in some communications.


### 2.Trigger the bug


+ Running environment
	* OS: Ubuntu (e.g., Ubuntu 18.04, 16.04, 22.04)

+ Source code required
	* `cJSON.c`, `cJSON.h` from https://github.com/DaveGamble/cJSON and a `test.c` (see below)

+ Steps

	+ Prepare a `test.c` file
		```
		#include "cJSON.h"
		#include <stdio.h>

		int main() {

			char buf[6];
			buf[0] = '\"'; // begain with quotation mark 
			buf[1] = '1';
			buf[2] = '\x0a'; // this character is not escaped! 
			buf[3] = '3';
			buf[4] = '\"'; // end with quotation mark
			buf[5] = 0;
		
			cJSON *root = NULL;
			root = cJSON_Parse(buf);
			if (!root) {
				const char *error_ptr = cJSON_GetErrorPtr();
				printf("error in json data:%s\n", error_ptr);
			} else {
				printf("No error, input type:%d\n", root->type);
				printf("However, this is unexpected!\n");
			}
		}
		```
		
	+ File structure (they are in the same directory)
		```
		- cJSON.c
		- cJSON.h
		- test.c
		```
	+ Compile and Run
		```
	 	clang test.c cJSON.c -o test
	 	./test

		```
		We will see it reports no error, which is unexpected. If you use other parsers, such as [github.com/nlohmann/json][3], which will correctly report an error. 



[1]: https://www.rfc-editor.org/rfc/rfc7159
[2]: https://github.com/simdjson/simdjson
[3]: https://github.com/nlohmann/json


