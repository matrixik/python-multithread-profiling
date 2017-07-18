* [Yappi](#yappi)
* [pprofile](#pprofile)
* [pyinstrument](#pyinstrument)
* [Pyflame](#pyflame)
* [Viewing profiling output files](#viewing-profiling-output-files)

# Yappi
https://bitbucket.org/sumerc/yappi/

`pip install yappi`
```python
def init_yappi():
  OUT_FILE = '/tmp/profile.out'

  import atexit
  import yappi

  print('[YAPPI START]')
  yappi.set_clock_type('wall')
  yappi.start()

  @atexit.register
  def finish_yappi():
    yappi.stop()

    print('[YAPPI WRITE]')

    stats = yappi.get_func_stats()

    # 'ystat' is Yappi internal format
    for stat_type in ['pstat', 'callgrind']:
      print('writing {}.{}'.format(OUT_FILE, stat_type))
      stats.save('{}.{}'.format(OUT_FILE, stat_type), type=stat_type)

    print('\n[YAPPI FUNC_STATS]')

    print('writing {}.func_stats'.format(OUT_FILE))
    with open('{}.func_stats'.format(OUT_FILE), 'wb') as fh:
      stats.print_all(out=fh)

    print('\n[YAPPI THREAD_STATS]')

    print('writing {}.thread_stats'.format(OUT_FILE))
    tstats = yappi.get_thread_stats()
    with open('{}.thread_stats'.format(OUT_FILE), 'wb') as fh:
      tstats.print_all(out=fh)

    print('[YAPPI DONE]')
```
Call `init_yappi()` in main or where needed (code [source](https://github.com/pantsbuild/pants/wiki/Debugging-Tips:-multi-threaded-profiling-with-yappi)).

# pprofile
https://github.com/vpelletier/pprofile

```bash
$ pprofile prog.py arg1 ...
$ pprofile --format callgrind --out profile.out.callgrind prog.py arg1 ...
$ pprofile --threads 0 prog.py arg1 ...
$ pprofile --statistic .01 prog.py arg1 ...
```

# pyinstrument
https://github.com/joerick/pyinstrument

```bash
pip install pyinstrument
python -m pyinstrument --html --outfile=profile.out.html prog.py arg1 ...
pyinstrument --html --outfile=profile.out.html prog.py arg1 ...
```

# Pyflame
https://github.com/uber/pyflame

It profile an already running process.

Install on Ubuntu:
```bash
$ sudo apt-add-repository ppa:trevorjay/pyflame
$ sudo apt update
$ sudo apt install pyflame
```

Or build:
```bash
# On Ubuntu
$ sudo apt install -y autoconf automake autotools-dev g++ pkg-config python-dev python3-dev libtool make
# On SLES
$ sudo zypper install -y autoconf automake gcc-c++ pkg-config python-devel python3-devel libtool make
$ git clone https://github.com/uber/pyflame.git && cd pyflame
$ ./autogen.sh && ./configure && make install
```

Get [**FlameGraph**](https://github.com/brendangregg/FlameGraph)
```bash
$ wget https://raw.githubusercontent.com/brendangregg/FlameGraph/master/flamegraph.pl && chmod 764 flamegraph.pl
```

Run it on already running process:
```bash
# Attach to PID 768 and profile it for 5 seconds
$ pyflame -s 5 --threads 768
$ pyflame -x -s 5 -o prof.pyflame.txt --threads -t prog.py arg1 ...
```

Create flame graph:
```bash
$ ./flamegraph.pl prof.pyflame.txt > prof.pyflame.svg
$ pyflame 12345 | flamegraph.pl > myprofile.svg

# Find only specific function
$ grep cpuid prof.pyflame.txt | ./flamegraph.pl > cpuid.svg
```

# Viewing profiling output files

View `pstat` with [**cprofilev**](https://github.com/ymichael/cprofilev)
```bash
$ pip install cprofilev
$ cprofilev -f profile.out.pstat

# You can create this file with cProfile
$ python -m cProfile -o profile.out.pstat prog.py arg1 ...
```
View `callgrind` files with **KCachegrind**
```bash
$ sudo apt install KCachegrind
$ kcachegrind profile.out.callgrind
```
