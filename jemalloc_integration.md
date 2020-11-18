# compile and install
```bash
git clone https://github.com/jemalloc/jemalloc
./autogen.sh
./configure --prefix=/xxx/lib64/jemalloc --with-jemalloc-prefix=je_
now if we want to make the debug build, change the Makefile, remove the O3
make -j 20
make install
```

We then can copy the head and lib to our project.

Note that
