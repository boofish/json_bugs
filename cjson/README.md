# cJSON rejects a valid text

### 1.Introduction

This is a bug related to Expected Behavior Violation.

Specifically, as documented in the [specification][1] (i.e, "A JSON parser MUST accept all texts that conform to the JSON grammar"),

json parser is expected to accept all grammar-valid texts.

However, we found a case that cJSON fails to accept it, and we have checked the text against the JSON grammar specified in RFC 7159 and found it to be compliant.
Other JSON parsers, such as those found at jsonlint.com and github.com/nlohmann/json, were able to parse the text without issue.

*potential effect*: This may lead to some unexpected errors, e.g., data out-of-sync in some communications, or invalid data 


### 2.Trigger the bug


+ Running environment
	* OS: Ubuntu (e.g., Ubuntu 18.04, 16.04, 22.04)

+ Source code required
	* `cJSON.c`, `cJSON.h` from https://github.com/DaveGamble/cJSON and a `test.c` (see below)

+ Steps

	+ Prepare a `test.c` file
		```
		#include <stdio.h>
		#include "cJSON.h"

		int main(int argc, char **argv)
		{
		    cJSON *root = NULL;
		    const char *s = "{\"a\": true, \"b\": [ null,9999999999999999999999999999999999999999999999912345678901234567]}";
		    root = cJSON_Parse(s);
		    if (root == NULL) {
		        const char *error_ptr = cJSON_GetErrorPtr();
		        printf("error in json data:%s\n", error_ptr);
		        return -1;
		    }
		    printf("parsed type:%d\n", root->type);  
		    cJSON_Delete(root);
		    return 0; 
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
		We will see it reports an error, which is unexpected. If you provide other valid json test, e.g., `const char *s = "{\"a\": true}";`, it will correctly parse it. 



[1]: https://www.rfc-editor.org/rfc/rfc7159


