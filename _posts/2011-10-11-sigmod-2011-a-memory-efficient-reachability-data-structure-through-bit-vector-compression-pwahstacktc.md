---
layout: post
title:  "SIGMOD 2011: A memory efficient reachability data structure through bit vector compression (PWAHStackTC)"
permalink: /pwahstacktc
tag: research
tag: pwahstacktc
---

Back in 2010, I worked on a research project at the University of Oxford with Prof. Oege de Moor. The project's aim was to efficiently compute, store, and answer reachability queries on large directed graphs. This work eventually led to a paper at ACM SIGMOD 2011, one of the largest international conferences on management of data. In this post, I will endeavour to convey the intuition behind our approach: bit vector compression using a specially designed scheme called PWAH.

The SIGMOD paper [<a href="#VanSchaik2011">VanSchaik2011</a>] can be downloaded from the ACM digital library (provided that you have access to that). The DOI is <a href="http://dx.doi.org/10.1145/1989323.1989419">10.1145/1989323.1989419</a>. The MSc. thesis on which this paper is based can be downloaded from the references [<a href="#VanSchaik2010">VanSchaik2010</a>]. A copy of the source code of our PWAH implementation and the implementation of Nuutila's algorithm is available on GitHub: [github.com/sj/pwahstacktc](https://github.com/sj/pwahstacktc).

## Some background: Tarjan's algorithm
In 1972, Robert Tarjan showed [<a href="#Tarjan1972">Tarjan1972</a>] how to use a depth first search to find the strongly connected components in a directed graph. This algorithm was extended by Esko Nuutila in the 1990's [<a href="#Nuutila1995">Nuutila1995</a>] to store reachability information of strongly connected components at the same time.

The algorithm introduced by Nuutila depends on an underlying data structure to <em>store</em> and (very important!) <em>merge</em> reachability information. Examples of such data structures include, but are not limited to:
<ul>
	<li>Array of integers (indices of strongly connected components)</li>
	<li>Bit vectors (each 1-bit indicates a reachable component)</li>
	<li>Interval lists</li>
</ul>

## Adding bit vector compression
### Introduction
In our experiments, bit vectors provided (by far) the best performance when merging reachability information of two strongly connected components. This can be done by computing the bitwise OR of the two bit vectors. However, storing a <em>O(n)</em> size bit vector for all <em>n</em> strongly connected components (worst case: <em>n = |V|</em>, the number of vertices) is equivalent to constructing (little over half) an <em>n</em> by <em>n</em> reachability matrix. This is considered infeasible for large graphs, <em>i.e.</em> storing reachability information for a graph with 1,000,000 vertices (and an equal number of strongly connected components) requires <em>(n * n + n) / 2 = 500,000,500,000</em> bits = over 58 gigabytes.

### Introducing PWAH compression
In order to reduce the memory usage of the bit vectors, whilst maintaining its main advantage (efficiency of merge operations), we decided to design a bit vector compression technique called Partitioned Word-Aligned Hybrid Compression (PWAH), based on Word-Aligned Hybrid Compression (WAH) by Wu et al. [<a href="#Wu2004">Wu2004</a>].

PWAH compresses bits by using 64-bit words which are split into 2 (PWAH-2), 4 (PWAH-4) or 8 (PWAH-8) partitions. The idea is most easily explained by an example PWAH-4 compressed 64-bits word:

&nbsp;
<table class="pwah">
<tbody>
<tr class="bits">
<td>1010</td>
<td>10000000001011</td>
<td>111010000100101</td>
<td>000000000010010</td>
<td>000000010000011</td>
</tr>
<tr class="descr">
<td>header</td>
<td>partition 1: <em>fill</em> of 11 blocks of 1-bits</td>
<td>partition 2: <em>literal</em> of 15 bits</td>
<td>partition 3: <em>fill</em> of 18 blocks of 0-bits</td>
<td>partition 4: <em>literal</em> of 15 bits</td>
</tr>
</tbody>
</table>
&nbsp;

The header and the four different partitions should be interpreted as follows:
<ul>
	<li>Header: 4 bits that indicate the type of each of the four partitions. A 1-bit in the header indicates a <em>fill</em> partition (see description of partition 1), a 0-bit indicates a <em>literal</em>partition (see description of partition 0).</li>
	<li>Partition 1: a <em>fill</em> partition (as indicated by the header): a long sequence of consecutive identical bits. The preceding 1-bit indicates that this partition represents a series of 1-bits, the other bits (<span class="bits">0...01011</span>) indicate the length of the consecutive series: 11 blocks = 165 bits (the block size of PWAH-4 equals 15 bits, see [<a href="#VanSchaik2010">VanSchaik2010</a>, <a href="#VanSchaik2011">VanSchaik2011</a>] for details).</li>
	<li>Partition 2: a <em>literal</em>partition (as indicated by the header): 15 bits which are not consecutive identical bits. These bits are included uncompressed.</li>
	<li>Partition 3: a <em>fill</em>partition representing 18 blocks of 0-bits (= 270 0-bits).</li>
	<li>Partition 4: another <em>literal</em> partition with 15 uncompressed bits.</li>
</ul>
The 64-bits PWAH compressed word represents a total of 465 uncompressed bits. Actively using some of the properties of a depth-first search, the compression scheme yields compressions of over a factor 6000 when applied to reachability information in strongly corrected components.

For a much more detailed description of PWAH in general, the different PWAH schemes, a theoretical and experimental analysis: see [<a href="#VanSchaik2010">VanSchaik2010</a>, <a href="#VanSchaik2011">VanSchaik2011</a>].

## Paper errata
Errata for [<a href="#VanSchaik2011">VanSchaik2011</a>]:
<ul>
<li>In Figure 2 (&ldquo;<em>WAH compression example</em>&rdquo;) on page 4, the last word of the WAH compressed bit vector is <em>not</em> a <em>0-fill</em>, but a <em>1-fill</em></li>
</ul>


## References
In order of appeareance:
<ul>
	<li><a name="VanSchaik2010"></a>VanSchaik2010: Sebastiaan J. van Schaik. Answering reachability queries on large directed graphs, introducing a new data structure using bit vector compression. MSc. thesis, Utrecht University, 2010. <a href="/wp-content/uploads/2011/10/INF-SCR-10-10.pdf">Download from here</a>.</li>
	<li><a name="VanSchaik2011"></a>VanSchaik2011: Sebastiaan J. van Schaik and Oege de Moor. A memory efficient reachability data structure through bit vector compression. In SIGMOD '11: Proceedings of the 37th SIGMOD international conference on Management of data, pages 913-924, New York, NY, USA, 2011. ACM. DOI: <a href="http://dx.doi.org/10.1145/1989323.1989419">10.1145/1989323.1989419</a>.</li>
	<li><a name="Tarjan1972"></a>Tarjan1972: Robert E. Tarjan. Depth-first search and linear graph algorithms. SIAM Journal on Computing, 1(2):146{160, 1972.</li>
	<li><a name="Nuutila1995"></a>Nuutila1995: Esko Nuutila. Efficient Transitive Closure Computation in Large Digraphs. PhD thesis, Finnish Academy of Technology, 1995. http://www.cs.hut.fi/~enu/tc.html.</li>
	<li><a name="Wu2004"></a>Wu2004: Kesheng Wu, Ekow J. Otoo, and Arie Shoshani. An efficient compression scheme for bitmap indices. Technical report, ACM Transactions on Database Systems, 2004.</li>
	<li><a name="Jin2008"></a>Jin2008: R. Jin, Y. Xiang, N. Ruan and H. Wang. Efficiently answering reachability queries on very large directed graphs. In SIGMOD '08: Proceedings of the ACM
SIGMOD international conference on Management of data, pages 595{608, New York, NY, USA, 2008.</li>
	<li><a name="Jin2009"></a>Jin2009: R. Jin, Y. Xiang, N. Ruan and D. Fuhry. 3-HOP: a high-compression indexing scheme for reachability query. In SIGMOD '09: Proceedings of the ACM SIGMOD international conference on Management of data, pages 813{826, New York, NY, USA, 2009.</li>
</ul>

## Source code


The implementation is licensed under the GPL:
<div style="margin-left: 50px; font-family: mono;">This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
GNU General Public License for more details.You should have received a copy of the GNU General Public License
along with this program. If not, see <a href="http://www.gnu.org/licenses/">http://www.gnu.org/licenses/</a></div><br/>

The following guide will help you download, compile and run the PWAH code on a 64-bit Linux machine. If you are a Windows user, your best bet is to install Eclipse (with the CDT plugin for C++ development) and import the PWAH Eclipse project into your workspace.

The C++ source code (including a Makefile and an Eclipse CDT project specification) of the implementation is available through a publicly accessible Git repository. If you have Git (see <a href="http://www.git-scm.com">the Git website</a>) installed, you can run the following command to download the source code (and datasets) into a directory called "PWAHStackTC":
<div style="margin-left: 50px;"><pre>git clone https://github.com/sj/PWAHStackTC</pre></div>
If you don't like using Git and would rather have a ZIP-file with the most recent implementation, visit <a href="https://github.com/sj/PWAHStackTC">the public GitHub repository</a> and click on &quot;Download ZIP&quot;.

Compilation instructions can be found in the provided README.txt file, which has been tested with the GNU C++ compiler (versions 4.5, 4.6, 4.7). Should you encounter any problems compiling or running PWAHStackTC, feel free to open a GitHub issue. After having successfully compiled the source code in the &quot;Release&quot; directory, invoke PWAHStackTC with the command line parameter "<code style='font-family: "Courier New", Monaco, monospace'>--help</code>" (that's "help" preceded by two dashes) for usage information. Simple usage example on an extremely small graph (10 vertices, included in the source code):
<div style="margin-left: 50px;">
<pre>$ ./PWAHStackTC --filename=../../datasets/nuutila.graph
Parsing graph file: /tmp/nuutila.graph... done, that took 0.1780 msecs
Number of vertices: 10, number of edges: 17
Computing transitive closure using PWAHStackTC -- not indexed&gt; (ALWAYS using multi-OR)... done, that took 0.0690 msecs
Number of components (vertices in condensation graph): 4
Counting number of edges in condensed transitive closure... 7 edges
Counting number of edges in transitive closure... 62 edges
Memory used by the reachability data structures of the PWAHStackTC -- not indexed&gt; algorithm: 192 bits
Memory used by equivalent interval lists: 192 bits
Total memory used by the PWAHStackTC -- not indexed&gt; algorithm: 512 bits

Generating 1000000 random sources and destinations... done, that took 53.3270 msecs
Performing 1000000 random queries... done, that took 22.7400 msecs
530326 pairs turned out to be reachable.
Number of bits required to store WAH compressed bitsets: 512
Average construction time over 1 runs: 0.0690 msecs
Average query time over 1 runs: 22.7400 msecs for 1000000 queries
AVG_CONSTR_TIME=0.0690
AVG_QUERY_TIME=22.7400
MEM_USAGE=512</pre>
</div>

PWAHStackTC expects its input graphs in the following format:
<ul>
	<li>line 1: the number of vertices <em>n</em> and the number of edges <em>m</em>, separated by a space</li>
	<li>following <em>n</em> lines: adjacency lists for vertex <em>n</em>, listing the indices of adjacent vertices. Note that indices start at 1. A vertex without any adjacent vertices is encoded as an empty line.</li>
</ul>


### Data sets
Data sets from multiple sources have been used for this paper, most noticeably from [<a href="#Jin2008">Jin2008</a>] and [<a href="#Jin2009">Jin2009</a>] and data sets kindly provided by Semmle Ltd. The graph files (graph format described above) are included in the Git repository.

<h3>Common compile-time and run-time errors</h3>
<h5>Compiler error: &ldquo;<em>invalid suffix (...) on integer constant</em>&rdquo;</h5>
<pre>(...)
Building file: ../src/datastructures/bitsets/wah/WAHBitSet.cpp
Invoking: GCC C++ Compiler
g++ -O3 -Wall -c -fmessage-length=0 -MMD -MP -MF"src/datastructures/bitsets/wah/WAHBitSet.d" -MT"src/datastructures/bitsets/wah/WAHBitSet.d" -o "src/datastructures/bitsets/wah/WAHBitSet.o" "../src/datastructures/bitsets/wah/WAHBitSet.cpp"
In file included from ../src/datastructures/bitsets/wah/WAHBitSet.cpp:26:
../src/datastructures/bitsets/wah/WAHBitSet.h:53:32: error: invalid suffix "b01111111111111111111111111111111" on integer constant
../src/datastructures/bitsets/wah/WAHBitSet.h:56:34: error: invalid suffix "b00000000000000000000000000000000" on integer constant
../src/datastructures/bitsets/wah/WAHBitSet.h:59:35: error: invalid suffix "b11000000000000000000000000000000" on integer constant
../src/datastructures/bitsets/wah/WAHBitSet.h:62:36: error: invalid suffix "b10000000000000000000000000000000" on integer constant
(...)</pre>
This means you're using a GCC version older than 4.3. Older GCC versions don't support integer constants written in binary format, see: <a href="http://gcc.gnu.org/gcc-4.3/changes.html" target=_blank>http://gcc.gnu.org/gcc-4.3/changes.html</a>. At the time of writing, GCC 4.3 has been available for more than five years. Please upgrade your compiler.

<h5>Compiler error: &ldquo;<em>this implementation depends on 64-bit longs, your platform does not seem to offer those. Aborting.</em>&rdquo;</h5>
This message is fairly self-explanatory: you're trying to compile the source code on a 32-bit operating system, but both the theory behind PWAH requires 64-bit data types. If you do have a 64-bit processor, you will have to upgrade your operating system to its 64-bit equivalent. If you have an old 32-bit hardware architecture, the only solution is buying a new machine. Sorry.

<h3><a id="contactme">Contact me</a></h3>
If you have any problems downloading, compiling, or using the implementation, please open [a GitHub issue in the repository](https://github.com/sj/PWAHStackTC/issues).
