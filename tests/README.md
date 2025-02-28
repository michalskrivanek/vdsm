# Vdsm tests

This tests suite is built on tox, pytest and nose. For more info on
these projects see:

- tox: https://tox.readthedocs.io/
- pytest: https://pytest.readthedocs.io/
- nose: http://nose.readthedocs.io/

# Tox and pytest based tests

Note: Not all tests have been converted to tox and/or pytest yet.

## Running the tests

Before running Vdsm tests, activate the virtual environment (which you need to
create once, with `make venv`, as described in
[/doc/development.md](/doc/development.md)):

    source ~/.venv/vdsm/bin/activate

When done with testing, you can deactivate the virtual environment:

    deactivate

To run tests from only one module, for example `lib`:

    tox -e lib

The test verbosity can be increased. Any arguments after `--` are passed
through to pytest:

    tox -e lib -- -vv

Before running storage tests, make sure you have set up the user storage
accordingly. To create storage you can do:

    make storage

See
[Setting up user storage](./storage/README.md#setting-up-user-storage)
for more information.

Run storage tests:

    tox -e storage -- -vv --no-cov storage

Note that for some test modules (like `storage` or `virt`), _if_ there
are any extra pytest arguments (after the `--`) then the search path at
the end needs to be specified explicitly, because the configuration in
`tox.ini` doesn't take care of it in that case and unintended tests from
other tox environments would be collected and run too.  
Here we also disabled the coverage check, which is configured in
`tox.ini`, because running subsets of the tests can't meet the desired
coverage goal anyway and will just produce a useless warning. In this case
some tests can only run as root. The same applies when we run the tests
from just one file. But note that you also don't get any coverage reports
in that case.

You can also use environment variables to pass options to pytest, then you
don't have to worry about the search path:

    export PYTEST_ADDOPTS="--no-cov -vv"
    tox -e storage

To run the tests from just a single file:

    tox -e storage -- --no-cov storage/image_test.py

Alternatively, all tests but storage tests can be run by doing:

    make tests

Furthermore, storage-only tests can be run by doing:

    make tests-storage


## Using tox virtual environment

Tox creates a virtual environment for each testenv. You don't have to
know anything about this if you run the tests with tox, but if you like
to use pytest directly, you want to use the pytest version installed in
the virtual environment.

To use tox virtual environment, activate it:

    [user@fedora vdsm (storage-tests)]$ source .tox/storage/bin/activate

Note that your shell prompt has changed, showing the virtual environment
name:

    (storage) [user@fedora vdsm (storage-tests)]$

Now ``pytest`` will run .tox/storage/bin/pytest. Lets run one test
module for example:

    (storage) [user@fedora vdsm (storage-tests)]$ cd tests

    (storage) [user@fedora vdsm (storage-tests)]$ export PYTHONPATH=../lib

    (storage) [user@fedora vdsm (storage-tests)]$ pytest storage/blockdev_test.py
    ============================= test session starts ==============================
    platform linux -- Python 3.6.8, pytest-4.6.5, py-1.9.0, pluggy-0.13.1
    rootdir: /home/pdm/ovirt/vdsm/vdsm-master, inifile: tox.ini
    plugins: cov-2.8.1
    collected 9 items

    storage/blockdev_test.py ssss....s                                       [100%]

    =========================== short test summary info ============================
    SKIPPED [1] tests/storage/blockdev_test.py:60: requires root
    SKIPPED [1] tests/storage/blockdev_test.py:79: requires root
    SKIPPED [2] tests/storage/blockdev_test.py:95: requires root
    SKIPPED [1] tests/storage/blockdev_test.py:155: requires root
    ===================== 4 passed, 5 skipped in 0.56 seconds ======================

When you finished with testing with the virtual environment, you can
deactivate it:

    (storage) [user@fedora tests (storage-tests)]$ deactivate

Note that your prompt has changed again:

    [user@fedora tests (storage-tests)]$


## Troubleshooting

Tox may fail due to an outdated or corrupted virtual environment.
To solve this you can recreate a specific tox environment without test execution:

    tox -e storage -r --notest

Or just remove the tox hidden folder to clean all tox environments by doing:

    rm -rf .tox


## Using pytest directly

Running all the tests takes couple of minutes. For quicker feedback you can use
pytest directly to run a module or some tests in a module.

You should use the pytest version from tox virual environmnet, please see
"Using tox virtual environment".

To use pytest directly, you must set the python path properly:

    export PYTHONPATH=../lib

Running entire module:

    pytest storage/misc_test.py

Running only one class:

    pytest storage/misc_test.py::TestSamplingMethod

Running only single test:

    pytest storage/misc_test.py::TestSamplingMethod::test_single_thread

Running all tests matching a pattern:

    pytest -k test_thread storage/misc_test.py

Running without slow and stress tests, using markers:

    pytest -m "not (slow or stress)" storage

Running only stress tests, using markers:

    pytest -m "stress" storage

## Running the tests as root

When using sudo, use env to pass PYTHONPATH to the underlying program:

    sudo env PYTHONPATH=$PYTHONPATH  tox ...

When running the tests as root, use another --basetemp directory for pytest:

    sudo ... pytest --basetemp=/var/tmp/vdsm-root storage/blockdev_test.py

This will avoid failures when you run the tests later as regular user, when
pytest will try to remove old temporary directories created by root.

# Nose based tests

## Running the tests

Running the tests from the tests directory, using vdsm code from the source
directory (lib/vdsm):

    ./run_tests_local.sh *.py

Running individual test module, test cases class or test function:

    ./run_tests_local.sh foo_test.py
    ./run_tests_local.sh foo_test:TestBar
    ./run_tests_local.sh foo_test:TestBar.test_baz

## Enabling slow tests

Some tests are too slow to run on each build. these are marked in the source
with the @slowtest decorator and are disabled by default.

To enable slow tests:
     
    ./run_tests_local.sh --enable-slow-tests filename [...]

Slow tests are also enabled if NOSE_SLOW_TESTS environment variable is set.

## Enabling stress tests

Some tests stress the resources of the system running the tests. These tests
are too slow to run on each build, and may fail on overloaded system or when
running tests in parallel. These tests are marked in the source with the
@stresstest decorator and are disabled by default.

To enable stress tests:
     
    ./run_tests_local.sh --enable-stress-tests filename [...]

Stress tests are also enabled if NOSE_STRESS_TESTS environment variable is set.

## Enabling threads leak check

To find tests leaking threads, you can enable the thread leak checker plugin:

    ./run_tests_local.sh --with-thread-leak-check filename [...]

To run the entire test suit with thread leak detection:

    make check NOSE_WITH_THREAD_LEAK_CHECK=1

## Enabling process leak check

To find tests leaking child processes, you can enable the process leak checker
plugin:

    ./run_tests_local.sh --with-process-leak-check filename [...]

To run the entire test suit with process leak detection:

    make check NOSE_WITH_PROCESS_LEAK_CHECK=1

## Enabling file leak check

To find tests leaking file descriptors, you can enable the process leak checker
plugin:

    ./run_tests_local.sh --with-file-leak-check filename [...]

To run the entire test suit with file leak detection:

    make check NOSE_WITH_FILE_LEAK_CHECK=1

## Control verbose level

By default, tests run without verbose level 1.
To run with verbose output, set verbose level to 3.

To set verbose level:
    
    make check NOSE_VERBOSE=<VERBOSE LEVEL>

## Functional test suite

The functional test suite is designed to test a running vdsm instance.  To run
the full suite of functional tests from within the installed directory:
    
    ./run_tests.sh functional/*.py

## Test timeout

If TIMEOUT environment variable is set, and a test run is too slow, it is
aborted, and a backtrace of all threads is printed.

Example usage:

    TIMEOUT=600 ./run_tests_local.sh foo_test.py
