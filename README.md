# tribble-index

[![Generated with nod](https://img.shields.io/badge/generator-nod-2196F3.svg?style=flat-square)](https://github.com/diegohaz/nod)
[![Coverage Status](https://img.shields.io/codecov/c/github/GMOD/tribble-index/master.svg?style=flat-square)](https://codecov.io/gh/GMOD/tribble-index/branch/master)
[![NPM version](https://img.shields.io/npm/v/@gmod/tribble-index.svg?logo=npm&style=flat-square)](https://www.npmjs.com/package/@gmod/tribble-index)
[![Build Status](https://img.shields.io/travis/rbuels/tribble-index/master.svg?logo=travis&style=flat-square)](https://travis-ci.org/rbuels/tribble-index)
[![Greenkeeper badge](https://badges.greenkeeper.io/GMOD/tribble-index-js.svg)](https://greenkeeper.io/)

Read htsjdk Tribble indexes (e.g. \*.vcf.idx files) using pure JavaScript. Supports only Tribble version 3 linear indexes right now.

## Install

    $ npm install --save tribble-index

## Usage

```js
import TribbleIndexedFile from '@gmod/tribble-index'
// or without ES6
const { TribbleIndexedFile } = require('@gmod/tribble-index')

const indexedFile = new TribbleIndexedFile({ path: 'path/to/data.vcf' })
// by default, assumes tribble index at path/to/data.vcf.idx
// can also provide `tribblePath` if the tribble is named differently

async function doStuff() {
  // iterate over lines in the specified region, each of which
  // is structured as
  const lines = []
  await indexedFile.getLines('contigA', 3000, 3200, line => lines.push(line))
  // lines is now an array of strings, which are data lines.
  // commented (meta) lines are skipped.
  // line strings do not include any trailing whitespace characters.
  console.log(lines)
  // ↳ [ 'contigA\t3000\trs17883296\tG\tT\t100\tPASS\tNS=3;DP=14;AF=0.5;DB;H2\tGT:AP\t0|0:0.000,0.000']

  // get the approximate number of data lines in the file for the
  // given reference sequence, excluding header, comment, and whitespace lines
  console.log(await indexedFile.lineCount('contigA'))
  // ↳ 109

  // get the "header text" string from the file, which is the first contiguous
  // set of lines in the file that all start with a "meta" character (usually #)
  console.log(await indexedFile.getHeader())
  // ↳ ##fileformat=VCFv4.1
  // ↳ #CHROM	POS	ID	REF	ALT	QUAL	FILTER	INFO	FORMAT	HG00096

  // or if you want a buffer instead, there is getHeaderBuffer()
  console.log(await indexedFile.getHeaderBuffer())
  // ↳ <Buffer 23 23 66 69 6c 65 66 6f 72 6d 61 74 3d 56 43 46 76 34 2e 31 0a 23 23 66 69 6c 65 ... >
}
```

Also supports a lower-level interface

```js
import fs from 'fs'
import read from '@gmod/tribble-index'
// or without ES6
const fs = require('fs')
const read = require('@gmod/tribble-index')

fs.readFile('path/to/data.vcf.idx', (err, buffer) => {
  if (err) throw err
  const index = read(buffer)

  console.log(index)
  const blocks = index.getBlocks('contigA', 3000, 3200)

  // can now use these blocks from the index to read the file
  // regions of interest
  fs.open('path/to/data.vcf', 'r', (err2, fd) => {
    if (err2) throw err2
    blocks.forEach(({ length, offset }) => {
      const buffer2 = Buffer.alloc(length)
      fs.read(fd, buffer2, 0, length, offset, (err3, bytesRead, buffer3) => {
        if (err3) throw err3
        console.log('got file data', buffer3)
      })
    })
    fs.close(fd, err4 => {
      if (err4) throw err4
    })
  })
})
```

## API

<!-- Generated by documentation.js. Update this documentation by updating the source code. -->

#### Table of Contents

-   [getBlocks](#getblocks)
    -   [Parameters](#parameters)
-   [getMetadata](#getmetadata)
-   [hasRefSeq](#hasrefseq)
    -   [Parameters](#parameters-1)
-   [read](#read)
    -   [Parameters](#parameters-2)
-   [constructor](#constructor)
    -   [Parameters](#parameters-3)
-   [checkLine](#checkline)
    -   [Parameters](#parameters-4)
-   [getReferenceSequenceNames](#getreferencesequencenames)
-   [getHeaderBuffer](#getheaderbuffer)
-   [getHeader](#getheader)
-   [lineCount](#linecount)
    -   [Parameters](#parameters-5)

### getBlocks

Get an array of { offset, length } objects describing regions of the
indexed file containing data for the given range.

#### Parameters

-   `refName` **[string](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** name of the reference sequence
-   `start` **integer** start coordinate of the range of interest
-   `end` **integer** end coordinate of the range of interest

### getMetadata

Returns an object like { fileMD5 fileName fileSize fileTimestamp
firstDataOffset flags magic properties type version chromosomes}

### hasRefSeq

Return true if the given reference sequence is present in the index,
false otherwise

#### Parameters

-   `refName` **[string](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** 

Returns **[boolean](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** 

### read

Parse the index from the given Buffer object. The buffer must contain
the entire index.

#### Parameters

-   `input` **[Buffer](https://nodejs.org/api/buffer.html)** 

Returns **(LinearIndex | IntervalTreeIndex)** an index object supporting the `getBlocks` method

### constructor

#### Parameters

-   `args` **[object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)** 
    -   `args.path` **[string](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)?** 
    -   `args.filehandle` **filehandle?** 
    -   `args.tribblePath` **[string](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)?** 
    -   `args.tribbleFilehandle` **filehandle?** 
    -   `args.metaChar` **[string](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)?** character that denotes the beginning of a header line (optional, default `'#'`)
    -   `args.columnNumbers` **[object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)?** object like {ref start end} defining the (1-based) columns in
        the file. end may be set to -1 if not present. default {ref: 1, start: 2, end: -1} (optional, default `{ref:1,start:2,end:-1}`)
    -   `args.oneBasedClosed` **[boolean](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Boolean)?** whether the indexed file uses one-based closed coordinates.
        default false (implying zero-based half-open coordinates) (optional, default `false`)
    -   `args.chunkSizeLimit` **[number](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Number)?** maximum number of bytes to fetch in a single `getLines` call.
        default 2MiB (optional, default `2000000`)
    -   `args.yieldLimit` **[number](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Number)?** maximum number of lines to parse without yielding.
        this avoids having a large read prevent any other work getting done on the thread.  default 300 lines. (optional, default `300`)
    -   `args.renameRefSeqs` **[function](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/function)?** optional function with sig `string => string` to transform
        reference sequence names for the purpose of indexing and querying. note that the data that is returned is
        not altered, just the names of the reference sequences that are used for querying. (optional, default `n=>n`)
    -   `args.blockCacheSize` **[number](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Number)?** maximum size in bytes of the block cache. default 5MB (optional, default `5*2**20`)

### checkLine

#### Parameters

-   `regionRefName` **[string](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** 
-   `regionStart` **[number](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Number)** region start coordinate (0-based-half-open)
-   `regionEnd` **[number](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Number)** region end coordinate (0-based-half-open)
-   `line`  

Returns **[object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)** like `{startCoordinate, overlaps}`. overlaps is boolean,
true if line is a data line that overlaps the given region

### getReferenceSequenceNames

get an array of reference sequence names

reference sequence renaming is not applied to these names.

Returns **[Promise](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise)** for an array of string sequence names

### getHeaderBuffer

get a buffer containing the "header" region of
the file, which are the bytes up to the first
non-meta line

Returns **[Promise](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise)** for a buffer

### getHeader

get a string containing the "header" region of the
file, is the portion up to the first non-meta line

Returns **[Promise](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise)** for a string

### lineCount

return the approximate number of data lines in the given reference sequence
returns -1 if the reference sequence is not found

#### Parameters

-   `refSeq` **[string](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** reference sequence name

Returns **[Promise](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise)** for number of data lines present on that reference sequence

## Academic Use

This package was written with funding from the [NHGRI](http://genome.gov) as part of the [JBrowse](http://jbrowse.org) project. If you use it in an academic project that you publish, please cite the most recent JBrowse paper, which will be linked from [jbrowse.org](http://jbrowse.org).

## License

MIT © [Robert Buels](https://github.com/rbuels)
