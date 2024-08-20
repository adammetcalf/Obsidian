
There are loads and loads, including unordered maps, lists, regex expressions for pattern matching, time durations, bitsets and tuples to name just a few. 

Here are the ones that I am likely to use most frequently.
# **Strings**

A string is a class that wraps a lot of functionality around essentially a sequence of chars. There are a [lot of methods](https://cplusplus.com/reference/string/string/) that can be called such as comparing and obtaining the size.

```
#include <string>
std::string s = "Hello, World!";
```


# **Vectors**

A vector in c++ is a class that wraps up functionality for dynamic arrays. There are a [lot of methods](https://cplusplus.com/reference/vector/vector/) that may be called to assist with handling array functions.

```
#include <vector>
std::vector<int> vec = {1, 2, 3, 4};
```


# **Filepaths**
 Filepaths are a class that wrap up path functionality and abstract away the difficulty of handling char or, strings for file path purposes. Again, [there are many methods](https://en.cppreference.com/w/cpp/filesystem/path) available.
```
#include <filesystem>
std::filesystem::path p = "/usr/local";
```

