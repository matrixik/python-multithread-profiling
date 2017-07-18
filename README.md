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

View `pstat` with [**cprofilev**](https://github.com/ymichael/cprofilev)
```bash
$ pip install cprofilev
$ cprofilev -f profile.out.pstat

// You can create this file with cProfile
$ python -m cProfile -o profile.out.pstat some_python_executable arg1 ...
```
View `callgrind` files with **KCachegrind**
```bash
$ sudo apt install KCachegrind
$ kcachegrind profile.out.callgrind
```

# pprofile
https://github.com/vpelletier/pprofile

```bash
$ pprofile --format callgrind --out profile.out.callgrind some_python_executable arg1 ...
$ pprofile some_python_executable arg1 ...
```

