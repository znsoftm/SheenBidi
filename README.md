SheenBidi
=========
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Build Status](https://api.travis-ci.org/mta452/SheenBidi.svg?branch=master)](https://travis-ci.org/mta452/SheenBidi)
[![Coverage Status](https://coveralls.io/repos/github/mta452/SheenBidi/badge.svg?branch=master)](https://coveralls.io/github/mta452/SheenBidi?branch=master)

SheenBidi implements Unicode Bidirectional Algorithm available at http://www.unicode.org/reports/tr9. It is a sophisticated implementaion which provides the developers an easy way to use UBA in their applications.

Here are some of the advantages of SheenBidi.

* Object based.
* Optimized to the core.
* Designed to be thread safe.
* Lightweight API for interaction.
* Supports UTF-8, UTF-16 and UTF-32 encodings.

## API
<img src="https://user-images.githubusercontent.com/2664112/39660778-1a09e544-505f-11e8-851a-887c19b32348.png" width="320">
The above screenshot depicts a visual representation of the API on a sample text.

### SBAlgorithm
It provides bidirectional type of each code unit in source string. Paragraph boundaries can be quried from it as determined by rule [P1](https://www.unicode.org/reports/tr9/#P1). Individual paragraph objects can be created from it by explicitly specifying the base level or deriving it from rules [P2](https://www.unicode.org/reports/tr9/#P2)-[P3](https://www.unicode.org/reports/tr9/#P3).

### SBParagraph
It represents a single paragraph of text processed with rules [X1](https://www.unicode.org/reports/tr9/#X1)-[I2](https://www.unicode.org/reports/tr9/#I2). It provides resolved embedding levels of all the code units of a paragraph.

### SBLine
It represents a single line of text processed with rules [L1](https://www.unicode.org/reports/tr9/#L1)-[L2](https://www.unicode.org/reports/tr9/#L2). However, it provides reordered level runs instead of reordered characters.

### SBRun
It represents a sequence of characters which have the same embedding level. The direction of a run would be right-to-left, if its embedding level is odd.

### SBMirrorLocator
It provides the facility to find out the mirrored characters in a line as determined by rule [L4](https://www.unicode.org/reports/tr9/proposed.html#L4).

## Dependency
SheenBidi does not depend on any external library. It only uses standard C library headers ```stddef.h```, ```stdint.h``` and ```stdlib.h```.

## Configuration
The configuration options are available in `Headers/SBConfig.h`.

* ```SB_CONFIG_LOG``` logs every activity performed in order to apply bidirectional algorithm.
* ```SB_CONFIG_UNITY``` builds the library as a single module and lets the compiler make decisions to inline functions.

## Compiling
SheenBidi can be compiled with any C compiler. The best way for compiling is to add all the files in an IDE and hit build. The only thing to consider however is that if ```SB_CONFIG_UNITY``` is enabled then only ```Source/SheenBidi.c``` should be compiled.

## Example
Here is a simple example written in C11.
```c
#include <stdio.h>
#include <string.h>

#include <SheenBidi.h>

int main(int argc, const char * argv[]) {
    const char *bidiText = u8"یہ ایک )car( ہے۔";

    SBCodepointSequence sequence = { SBStringEncodingUTF8, (void *)bidiText, strlen(bidiText) };
    SBAlgorithmRef algorithm = SBAlgorithmCreate(&sequence);
    SBParagraphRef paragraph = SBAlgorithmCreateParagraph(algorithm, 0, sequence.stringLength, SBLevelDefaultLTR);
    SBLineRef line = SBParagraphCreateLine(paragraph, 0, sequence.stringLength);

    SBUInteger runCount = SBLineGetRunCount(line);
    const SBRun *runArray = SBLineGetRunsPtr(line);
    for (SBUInteger i = 0; i < runCount; i++) {
        printf("Run Level: %ld\n", (long)runArray[i].level);
        printf("Run Offset: %ld\n", (long)runArray[i].offset);
        printf("Run Length: %ld\n\n", (long)runArray[i].length);
    }

    SBMirrorLocatorRef locator = SBMirrorLocatorCreate();
    SBMirrorLocatorLoadLine(locator, line, sequence.stringBuffer);
    const SBMirrorAgentRef agent = SBMirrorLocatorGetAgent(locator);
    while (SBMirrorLocatorMoveNext(locator)) {
        printf("Mirror Location: %ld\n", (long)agent->index);
        printf("Mirror Code Point: %ld\n\n", (long)agent->mirror);
    }
    SBMirrorLocatorRelease(locator);

    SBLineRelease(line);
    SBParagraphRelease(paragraph);
    SBAlgorithmRelease(algorithm);
    
    return 0;
}
```
