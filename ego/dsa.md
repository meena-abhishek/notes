`'0' + i`  - int to char
`c - '0'`  - char to int
use `to_string()` for numbers $>9$ 

emplace_back is faster and can also push pair like `v.emplace_back(1,2)`
instead of `v.push_back({1,2})`
because it constructs object directly in the memory of vector instead of creating it somewhere else and moving it to vector.

custom comparator `comp(p1,p2)`, return true if they are in correct order i.e $p1, p2$

`__builtin_popcount()`
`*max_element(a,a+n)`

for longest consecutive sum, try to take sum of the left most (shortest) sub array as $sum-k$ , in case of zero's, don't update the sum index in hashmap if already exist.


https://leetcode.com/problems/number-of-ways-to-assign-edge-weights-ii/description/?envType=daily-question&envId=2026-06-12
https://chatgpt.com/share/6a2b907d-7760-83e9-b370-d736580413f1