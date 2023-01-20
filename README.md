<div id="header">
    <p align="center">
      <b>sqlite3.cmake</b><br>
  	  <span font-size="16px">a cmake implementation for sqlite3 for installing/using SQLite3</span><br>
      <span font-size="12px">Made by <a href="http://tek256.com">Devon</a> with love.</span><br><br>
      <span><img src="https://github.com/tek256/sqlite3.cmake/workflows/Build%20SQLite3/badge.svg"></span>
    </p>
</div>

### Current SQLite3 Version: 3.40.1

### About
A CMake implementation for installing/using SQLite3. Source code for SQLite3 is sourced from the [SQLite Amalgamation](https://www.sqlite.org/download.html).


### Installation (System Wide)

**NOTE: Make sure you have administrative privlidges before attempting to install**

```
git clone https://github.com/tek256/sqlite3.cmake
cd sqlite3.cmake
cmake -Bbuild -DCMAKE_BUILD_TYPE=Release
cmake --build build
cmake --install build
```


### Credit
Written in collaboration with [bruxisma](https://github.com/bruxisma). I was the monkey, they were the brains.

