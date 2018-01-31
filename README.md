========================
Using the RaaS Scheduler
========================

You can test the RaaS module using the sseapi-client directly from the Python
shell:

.. code-block:: python

    from sseapiclient.tornado import SyncClient
    client = SyncClient.connect('http://localhost:80', 'root', 'salt')
    client.api.schedule.get()


Adding Jobs
-----------
One or more scheduled jobs may be added at once. These jobs are passed in
using a dict called ``data``. Each key in the ``data`` dict is the name of the
job, and the value of that key is another dictionary containing the following
kwargs:

cmd
```
Required. This is the type of command that will be run. The following values
may be used:

* ``local``: A standard ``salt`` command to be executed across one or more minions.
* ``runner``: A runner command (``salt-run``), to be executed on the master.
* ``ssh``: A ``salt-ssh`` command.
* ``wheel``: A wheel command, for management of the master.

tgt
```
Required, if using a ``local`` command, and a ``tgt_uuid`` is not used. This is
a dict containing one or more masters as the key(s), and another dict containing
a ``tgt`` and ``tgt_type`` for each of those masters.  ``tgt_type`` is usually
set to ``glob``, but may be changed to ``pcre``, ``grain``, etc.

.. code-block:: python

    # Only one master
    'tgt': {
        'master1': {
            'tgt': 'flay',
            'tgt_type': 'glob',
        },
    }

    # Multiple masters
    'tgt': {
        'master1': {
            'tgt': '*',
            'tgt_type': 'glob',
        },
        'master2': {
            'tgt': '*',
            'tgt_type': 'glob',
        },
    }

    # All masters
    'tgt': {
        '*': {
            'tgt': '*',
            'tgt_type': 'glob',
        },
    }


tgt_uuid
````````
Required for ``local`` commands, if a ``tgt`` is not specified.

masters
```````
Required for ``runner`` and ``wheel`` commands. A list of masters which will
process the command.

fun
```
Required. This is the salt function (``file.touch``, ``manage.status``, etc)
that will be executed.

arg
```
Optional. A dict containing one or both of the following arguments.

arg
~~~
Optional. A list of any ``*args`` (positional arguments) to be used for the job.

kwarg
~~~~~
Optional. A dict containing any ``**kwargs`` (keyword arguments) to be used for
the job.

schedule
````````
A dict containing schedule-specific arguments, as described below.

seconds
~~~~~~~
The number of seconds between each iteration. Please note that this amount, as
well as ``minutes``, ``hours``, and ``days``, contingent upon the start time of
the SaltStack Enterprise service, rather than upon the clock time as would be
expected from a cron-style job.

The following will schedule the command: ``state.sls httpd test=True`` every
3600 seconds (every hour).

.. code-block:: python

    client.api.schedule.add({
        'testjob': {
            'cmd':'local',
            'fun': 'state.sls',
            'tgt': {
                'master1': {
                    'tgt': 'flay',
                    'tgt_type': 'glob',
                },
            },
            'arg': {
                'arg': ['httpd'],
                'kwarg': {'test': True},
            },
            'schedule': {
                'seconds': 3600,
            },
        }
    })

minutes
~~~~~~~
See seconds.

hours
~~~~~
See seconds.

days
~~~~
See seconds.

splay
~~~~~
The following will schedule the command: ``state.sls httpd test=True`` every
3600 seconds (every hour) splaying the time between 0 and 15 seconds.

.. code-block:: python

    client.api.schedule.add({
        'testjob': {
            'cmd': 'local',
            'fun': 'state.sls',
            'tgt': {
                'master1': {
                    'tgt': 'flay',
                    'tgt_type': 'glob',
                },
            },
            'arg': {
                'arg': ['httpd'],
                'kwarg': {'test': True},
            },
            'schedule': {
                'seconds': 3600,
                'splay': 15,
            },
        },
    })

The following will schedule the command: ``state.sls httpd test=True`` every
3600 seconds (every hour) splaying the time between 10 and 15 seconds.

.. code-block:: python

    client.api.schedule.add({
        'testjob': {
            'cmd':'local',
            'fun': 'state.sls',
            'tgt': {
                'master1': {
                    'tgt': 'flay',
                    'tgt_type': 'glob',
                },
            },
            'arg': {
                'arg': ['httpd'],
                'kwarg': {'test': True},
            },
            'schedule': {
                'seconds': 3600,
                'splay': {
                    'start': 10,
                    'end': 15,
                },
            },
        },
    })

Schedule by Date and Time
-------------------------
Frequency of jobs can also be specified using date strings supported by
the Python ``dateutil`` library.  The following will schedule the command
``state.sls httpd test=True`` at 5:00 PM, on the server running SaltStack
Enterprise.

.. code-block:: python

    client.api.schedule.add({
        'testjob': {
            'cmd':'local',
            'fun': 'state.sls',
            'tgt': {
                'master1': {
                    'tgt': 'flay',
                    'tgt_type': 'glob',
                },
            },
            'arg': {
                'arg': ['httpd'],
                'kwarg': {'test': True},
            },
            'schedule': {
                'when': '5:00pm',
            },
        },
    })

The following will schedule the command: ``state.sls httpd test=True`` at
5:00 PM on Monday, Wednesday and Friday, and 3:00 PM on Tuesday and Thursday.

.. code-block:: python

    client.api.schedule.add({
        'testjob': {
            'cmd':'local',
            'fun': 'state.sls',
            'tgt': {
                'master1': {
                    'tgt': 'flay',
                    'tgt_type': 'glob',
                },
            },
            'arg': {
                'arg': ['httpd'],
                'kwarg': {'test': True},
            },
            'schedule': {
                'when': [
                    'Monday 5:00pm',
                    'Tuesday 3:00pm',
                    'Wednesday 5:00pm',
                    'Thursday 3:00pm',
                    'Friday 5:00pm',
                ],
            },
        },
    })

The following will schedule the command: ``state.sls httpd test=True`` every
3600 seconds (every hour) between the hours of 8:00 AM and 5:00 PM. The
``range`` parameter must be a dictionary with the date strings using the
``dateutil`` format.

.. code-block:: python

    client.api.schedule.add({
        'testjob': {
            'cmd':'local',
            'fun': 'state.sls',
            'tgt': {
                'master1': {
                    'tgt': 'flay',
                    'tgt_type': 'glob',
                },
            },
            'arg': {
                'arg': ['httpd'],
                'kwarg': {'test': True},
            },
            'schedule': {
                'seconds': 3600,
                'range': {
                    'start': '8:00am',
                    'end': '5:00pm',
                },
            },
        },
    })

Using the ``invert`` option for ``range``, the following will schedule the
command ``state.sls httpd test=True`` every 3600 seconds (every hour) until the
current time is between the hours of 8:00 AM and 5:00 PM. The ``range``
parameter must be a dictionary with the date strings using the ``dateutil``
format.

.. code-block:: python

    client.api.schedule.add({
        'testjob': {
            'cmd':'local',
            'fun': 'state.sls',
            'tgt': {
                'master1': {
                    'tgt': 'flay',
                    'tgt_type': 'glob',
                },
            },
            'arg': {
                'arg': ['httpd'],
                'kwarg': {'test': True},
            },
            'schedule': {
                'seconds': 3600,
                'range': {
                    'invert': True,
                    'start': '8:00am',
                    'end': '5:00pm',
                },
            },
        },
    })

The following will schedule the function ``pkg.install`` to be executed once at
the specified time. The schedule entry ``job1`` will not be removed after the
job completes, therefore use ``schedule.delete`` to manually remove it
afterwards.

.. code-block:: python

    client.api.schedule.add({
        'testjob': {
            'cmd':'local',
            'fun': 'pkg.install',
            'tgt': {
                'master1': {
                    'tgt': 'flay',
                    'tgt_type': 'glob',
                },
            },
            'arg': {
                'kwarg': {
                    'pkgs': [{'bar': '>1.2.3'}],
                    'refresh': True,
                    },
            },
            'schedule': {
                'once': '2016-01-07T14:30:00',
            },
        },
    })

The default date format is ISO 8601 but can be overridden by also specifying
the ``once_fmt`` option, like this:

.. code-block:: python

    client.api.schedule.add({
        'testjob': {
            'cmd':'local',
            'fun': 'test.ping',
            'tgt': {
                'master1': {
                    'tgt': 'flay',
                    'tgt_type': 'glob',
                },
            },
            'schedule': {
                'once': '2015-04-22T20:21:00',
                'once_fmt': '%Y-%m-%dT%H:%M:%S',
            },
        },
    })

Maximum Parallel Jobs Running
-----------------------------
The scheduler also supports ensuring that there are no more than N copies of
a particular routine running. Use this for jobs that may be long-running
and could step on each other or pile up in case of infrastructure outage.

The default for ``maxrunning`` is 1.

.. code-block:: python

    client.api.schedule.add({
        'testjob': {
            'cmd':'local',
            'fun': 'test.sleep',
            'tgt': {
                'master1': {
                    'tgt': 'flay',
                    'tgt_type': 'glob',
                },
            },
            'arg': {
                'arg': [3600],
            },
            'schedule': {
                'jid_include': True,
                'maxrunning': 1,
            },
        },
    })

Note the presence of the ``jid_include`` argument, which will ensure that the
return data includes the JID of the job.

Cron-like Schedule
------------------
The scheduler also supports scheduling jobs using a cron like format.

.. code-block:: python

    client.api.schedule.add({
        'testjob': {
            'cmd':'local',
            'fun': 'state.sls',
            'tgt': {
                'master1': {
                    'tgt': 'flay',
                    'tgt_type': 'glob',
                },
            },
            'arg': {
                'arg': ['httpd'],
                'kwarg': {'test': True},
            },
            'schedule': {
                'cron': '*/15 * * * *',
            },
        },
    })

Job Data Return
---------------
By default, data about jobs runs from the Salt scheduler is returned to the
master. Setting the ``return_job`` parameter to ``False`` will prevent the data
from being sent back to the Salt master.

.. code-block:: python

    client.api.schedule.add({
        'testjob': {
            'cmd':'local',
            'fun': 'state.sls',
            'tgt': {
                'master1': {
                    'tgt': 'flay',
                    'tgt_type': 'glob',
                },
            },
            'arg': {
                'arg': ['httpd'],
                'kwarg': {'test': True},
            },
            'schedule': {
                'return_job': False,
            },
        },
    })

Job Metadata
------------
It can be useful to include specific data to differentiate a job from other
jobs. Using the ``metadata`` parameter special values can be associated with
a scheduled job. These values are not used in the execution of the job,
but can be used to search for specific jobs later if combined with the
``return_job`` parameter. The ``metadata`` parameter must be specified as a
dictionary, othewise it will be ignored.

.. code-block:: python

    client.api.schedule.add({
        'testjob': {
            'cmd':'local',
            'fun': 'state.sls',
            'tgt': {
                'master1': {
                    'tgt': 'flay',
                    'tgt_type': 'glob',
                },
            },
            'arg': {
                'arg': ['httpd'],
                'kwarg': {'test': True},
            },
            'schedule': {
                'return_job': False,
                'metadata': {'foo': 'bar'},
            },
        },
    })

Run on Start
------------
By default, any job scheduled based on the startup time of the minion will run
the scheduled job when the minion starts up. Sometimes this is not the desired
situation. Using the ``run_on_start`` parameter set to ``False`` will cause the
scheduler to skip this first run and wait until the next scheduled run:

.. code-block:: python

    client.api.schedule.add({
        'testjob': {
            'cmd':'local',
            'fun': 'state.sls',
            'tgt': {
                'master1': {
                    'tgt': 'flay',
                    'tgt_type': 'glob',
                },
            },
            'arg': {
                'arg': ['httpd'],
                'kwarg': {'test': True},
            },
            'schedule': {
                'seconds': 3600,
                'run_on_start': False,
            },
        },
    })

Until and After
---------------
Using the ``until`` argument, the scheduler allows you to specify an end time
for a scheduled job. If this argument is specified, jobs will not run once the
specified time has passed. Time should be specified in a format supported by
the ``dateutil`` library.

.. code-block:: python

    client.api.schedule.add({
        'testjob': {
            'cmd':'local',
            'fun': 'state.sls',
            'tgt': {
                'master1': {
                    'tgt': 'flay',
                    'tgt_type': 'glob',
                },
            },
            'arg': {
                'arg': ['httpd'],
                'kwarg': {'test': True},
            },
            'schedule': {
                'seconds': 15,
                'until': '12/31/2015 11:59pm',
            },
        },
    })

Using the ``after`` argument, the scheduler allows you to specify a start time
for a scheduled job.  If this argument is specified, jobs will not run until
the specified time has passed. Time should be specified in a format supported
by the ``dateutil`` library.

.. code-block:: python

    client.api.schedule.add({
        'testjob': {
            'cmd':'local',
            'fun': 'state.sls',
            'tgt': {
                'master1': {
                    'tgt': 'flay',
                    'tgt_type': 'glob',
                },
            },
            'arg': {
                'arg': ['httpd'],
                'kwarg': {'test': True},
            },
            'schedule': {
                'seconds': 15,
                'after': '12/31/2015 11:59pm',
            },
        },
    })


Getting Jobs
------------
The ``get()`` function is used to retrieve one or more jobs. If no arguments
are specified, then all jobs will be returned.

.. code-block:: python

    client.api.schedule.get()

One or more jobs may also be specified, as a list.

.. code-block:: python

    client.api.schedule.get(['testjob'])
    client.api.schedule.get(['testjob1', 'testjob2'])

Normally, jobs in the past will not be shown. To include those in the return,
set ``show_past`` to ``True``.

.. code-block:: python

    client.api.schedule.get(show_past=True)

Jobs may be queried, based on the information in ``arg`` and ``kwarg``. To do
so, ``arg`` and/or ``kwarg`` must be ``True``.

.. code-block:: python

    client.api.schedule.get(query='/tmp/somefile', arg=True)
    client.api.schedule.get(query='/tmp/somefile', kwarg=True)
    client.api.schedule.get(query='/tmp/somefile', arg=True, kwarg=True)

To return only jobs that are of a certain type of command (``local``,
``runner``, ``ssh``, or ``wheel``), use the ``cmd`` argument.

.. code-block:: python

    client.api.schedule.get(cmd='local')

To search by function name, use ``fun``.

.. code-block:: python

    client.api.schedule.get(fun='test.ping')

To search based on master, use ``masters``.

.. code-block:: python

    client.api.schedule.get(masters='master1')

To search based on a target, use ``tgt``:

.. code-block:: python

    client.api.schedule.get(tgt='web1')

To search for a schedule during a specific date range, specify a list of dates
in ISO 8601 format.Such a date string may look like: ``2017-11-27T13:00:00``.

.. code-block:: python

    client.api.schedule.get(
        daterange=['2017-11-27T13:00:00',
                   '2017-11-27T13:30:00']
    )

An alternate date format may be specified for the ``daterange`` argument as
``date_fmt``. The default (being ISO 8601) is ``%Y-%m-%dT%H:%M:%S``.

.. code-block:: python

    client.api.schedule.get(
        daterange=['2017-11-27T15:30:00', '2017-11-28T15:30:00'],
        date_fmt='%Y-%m-%dT%H:%M:%S',
    )

Normally a limit of 50 results is imposed. This may be changed with the
``limit`` argument.

.. code-block:: python

    client.api.schedule.get(limit=75)

To get a page of results past the first, specify ``page``.

.. code-block:: python

    client.api.schedule.get(limit=75, page=2)

Removing Jobs
-------------
The ``remove()`` function is used to delete a single job by name. Its only
argument is the name of the function to delete.

.. code-block:: python

    client.api.schedule.remove('testjob')


Searching Job History
---------------------
Job history may searched using the ``search_history()`` function. The most
basic search is for any jobs that were executed during a specific date range.

The daterange is a list of two ISO 8601 date strings. Such a date string may
look like: ``2017-11-27T13:00:00``.

To search the job history by date, use the ``daterange`` field, which expects
a list.

.. code-block:: python

    client.api.schedule.search_history(
        daterange=['2017-11-27T13:00:00',
                   '2017-11-27T13:30:00']
    )

As with other functions, the format of the daterange may be changed. This is
done using the ``date_fmt`` argument. The default (being an ISO 8601 string)
is ``%Y-%m-%dT%H:%M:%S``.

The data returned will be a paginated list of jobs which matched the date range.
By default the ``limit`` is set to ``50`` and the ``page`` is set to ``0``. To
perform the above search with the second page of 100 jobs, use:

.. code-block:: python

    client.api.schedule.search_history(
        daterange=['2017-11-27T13:00:00',
                   '2017-11-27T13:30:00']
        limit=100,
        page=1,
    )

These dates may also be specified in JID (Job ID) format, which is a 20 digit
integer.  If we were to look at a JID of ``20171127123418206477``, we would
find the following information concationated together:

* 2017 (YYYY)
* 11 (MM)
* 27 (DD)
* 12 (hour)
* 34 (minute)
* 18 (second)
* 206477 (microseconds)

The JIDs in this range don't need to contain all of these fields, but they do
still need to be 20 digits long. However, a zero-padded string will suffice.
For instance, ``20171127123418206477`` could be expresssed as
``20171127000000000000`` if you only needed a date of 2017-11-27.


Arguments for Searching History
-------------------------------
The following arguments are available for searching job history.

daterange
`````````
A list of two dates, in ISO 8601 format.

date_fmt
````````
An alternate format for the ``daterange``.

limit
`````
How many records to return at a time.

page
````
Which page of records to return. In database terms, ``page * limit`` is used
to generate the ``OFFSET`` of a query.

query
`````
A string of text to search for. This may also be a regular expression, but in
order to perform a case-insensitive search, this text will be lowercased, in
addition to any fields that will be searched.

name
````
Whether to search for the ``query`` string inside the name of the job.

arg
```
Whether to search for the ``query`` string inside a job's ``*args``.

kwarg
`````
Whether to search for the ``query`` string inside a job's ``**kwargs``.

cmd
```
One of ``local``, ``runner``, ``ssh``, ``wheel``, as discussed above.

fun
```
A string of text to search for inside function names. This is a contains-style
query, rather than an == query. For instance, searching for ``test`` will
return jobs matching both ``test.ping`` and ``test.sleep``.

tgt
```
A target string to search for. As with ``fun``, this is a contains-style search.

masters
```````
A string to look for in the name(s) of the masters that were assigned the job.
As with ``fun``, this is a contains-style search.


Search Examples
---------------
Following are some examples of history search calls.

.. code-block:: python

    client.api.schedule.search_history(
        query='testjob',
        name=True
    )

    client.api.schedule.search_history(
        query='somefile',
        arg=True,
        kwarg=True
    )

    client.api.schedule.search_history(
        fun='test.sleep',
        cmd='runner'
    )


Return Example
--------------
The return for the ``search_history()`` function will look like the following
(reformatted here for readability).

..code-block:: python

    RPCResponse(
        riq=4,
        ret=[
            {'data':
                {'arg': {'arg': ['/tmp/somefile'], 'kwarg': {}},
                'cmd': 'local',
                'fun': 'file.touch',
                'tgt': {'master1': {'tgt': 'glob', 'tgt_type': 'glob'}},
                '__pub_id': 'master',
                '__pub_fun': 'job.run',
                '__pub_jid': '20171113163107028356',
                '__pub_pid': 20321,
                '__pub_fun_args': [
                    {'arg': {'arg': ['/tmp/somefile'], 'kwarg': {}},
                    'cmd': 'local',
                    'fun': 'file.touch',
                    'tgt': {'master1': {'tgt': 'glob', 'tgt_type': 'glob'}}
                    }
                ],
                '__pub_schedule': 'testjob'}
            }
        ],
        error=None,
        warnings=[]
    )


Finding Future Jobs
-------------------
It is possible to query for jobs that are expected to execute in the future.
This is accomplished using the ``futures()`` function.

.. code-block:: python

    client.api.schedule.futures()

Arguments
`````````
This function accepts three types of arguments: zero or more jobs (by name),
a date range, and a limit. Jobs are specified using the ``jobs`` argument. A
date range can be specified using either the ``daterange`` argument, or the
``start`` and ``end`` arguments. Limiting is handled using the ``instances``
argument.

jobs
````
Optional. List. If used, this argument is a list of of jobs to query. By
default, all jobs will be queried.

.. code-block:: python

    client.api.schedule.futures(jobs=['testjob1', 'testjob2'])

daterange
`````````
Optional. List. If used, this is a list of timestamp strings, in ISO 8601
format. Such a timestamp would look like ``2017-11-27T15:30:00``.

.. code-block:: python

    client.api.schedule.futures(
        daterange=['2017-11-27T15:30:00', '2017-11-28T15:30:00']
    )

date_fmt
````````
Optional. String. If used, this specifies an alternate date format for the
``daterange`` argument. The default (being ISO 8601) is ``%Y-%m-%dT%H:%M:%S``.

.. code-block:: python

    client.api.schedule.futures(
        daterange=['2017-11-27T15:30:00', '2017-11-28T15:30:00'],
        date_fmt='%Y-%m-%dT%H:%M:%S',
    )

start
`````
Optional. Integer. If used, this is a timestamp. The default is <now>.

end
```
Optional. Integer. If used, this is a timestamp. The default is <now + 8 weeks>.

.. code-block:: python

    client.api.schedule.futures(start=1512504728, end=1512505728)

instances
`````````
Optional. Integer. By default, only the first 100 results will be returned.
If a different limit is desired, it can be specified using the ``instances``
argument.

.. code-block:: python

    client.api.schedule.futures(
        daterange=['2017-11-27T15:30:00', '2017-11-28T15:30:00'],
        instances=50
    )


Return Data for Futures
-----------------------
The preceding example will yield a result similar to:

.. code-block:: python

    RPCResponse(
        riq=9,
        ret=[{
            'job': 'testjob',
            'times': [
                1512505068
                1512505129
                1512505190
                1512505251
                1512505312
                1512505373
                1512505434
                1512505495
                1512505556
                1512505617
                1512505678
                1512505739
            ]
        }],
        error=None,
        warnings=[]
    )


Running a Job Once
------------------
There are two ways to specify that a job should run a single time. First, there
is the ``once`` option:

.. code-block:: python

    client.api.schedule.add({
        'testjob': {
            'cmd': 'local',
            'fun': 'file.touch',
            'tgt': {
                'master1': {
                    'tgt': 'flay',
                    'tgt_type': 'glob',
                },
            },
            'arg': {
                'arg': ['/tmp/someotherfile'],
            },
            'schedule': {
                'once': '2018-01-07T14:30:00',
            },
        },
    })

The format of ``once`` may be altered using the ``once_fmt`` option:

.. code-block:: python

    client.api.schedule.add({
        'testjob': {
            'cmd': 'local',
            'fun': 'file.touch',
            'tgt': {
                'master1': {
                    'tgt': 'flay',
                    'tgt_type': 'glob',
                },
            },
            'arg': {
                'arg': ['/tmp/someotherfile'],
            },
            'schedule': {
                'once': '2018-04-22T20:21:00',
                'once_fmt': '%Y-%m-%dT%H:%M:%S',
            },
        },
    })


Skipping Jobs
-------------
Skipping a job is almost the inverse of running a job a single time. A range
is specified, but explicit times within that range can be set aside to not run
the job. This is done using the ``skip_time`` option:

.. code-block:: python

    client.api.schedule.add({
        'testjob': {
            'cmd':'local',
            'fun': 'file.touch',
            'tgt': {
                'master1': {
                    'tgt': 'flay',
                    'tgt_type': 'glob',
                },
            },
            'arg': {
                'arg': ['/tmp/someotherfile'],
            },
            'schedule': {
                'hours': 3,
                'skip_time': '2018-04-22T17:00:00',
            },
        },
    })

The ``skip_time`` option may be a string (ISO 8601) or a list of strings.

.. code-block:: python

    client.api.schedule.add({
        'testjob': {
            'cmd':'local',
            'fun': 'file.touch',
            'tgt': {
                'master1': {
                    'tgt': 'flay',
                    'tgt_type': 'glob',
                },
            },
            'arg': {
                'arg': ['/tmp/someotherfile'],
            },
            'schedule': {
                'hours': 3,
                'skip_time': [
                    '2018-04-22T17:00:00',
                    '2018-04-22T20:00:00',
                ],
            },
        },
    })

A ``skip_fmt`` may be added. The default is ``%Y-%m-%dT%H:%M:%S``.

.. code-block:: python

    client.api.schedule.add({
        'testjob': {
            'cmd':'local',
            'fun': 'file.touch',
            'tgt': {
                'master1': {
                    'tgt': 'flay',
                    'tgt_type': 'glob',
                },
            },
            'arg': {
                'arg': ['/tmp/someotherfile'],
            },
            'schedule': {
                'hours': 3,
                'skip_time': '2018-04-22T17:00:00',
                'skip_fmt': '%Y-%m-%dT%H:%M:%S',
            },
        },
    })


Disabling Jobs
--------------
By default jobs are set to having ``enabled`` be equal to ``True``. To disable
a job schedule, explicitly set ``enabled`` to ``False``.

.. code-block:: python

    client.api.schedule.add({
        'testjob': {
            'cmd':'local',
            'fun': 'file.touch',
            'tgt': {
                'master1': {
                    'tgt': 'flay',
                    'tgt_type': 'glob',
                },
            },
            'arg': {
                'arg': ['/tmp/someotherfile'],
            },
            'schedule': {
                'days': 3,
                'enabled': False,
            },
        },
    })


Specifying Ranges
-----------------
Be default job schedules will run at the interval specified, no matter when that
interval occurs. It is possible to set a time range in which a job is allowed
to run using the ``range`` option.

.. code-block:: python

    client.api.schedule.add({
        'testjob': {
            'cmd':'local',
            'fun': 'file.touch',
            'tgt': {
                'master1': {
                    'tgt': 'flay',
                    'tgt_type': 'glob',
                },
            },
            'arg': {
                'arg': ['/tmp/someotherfile'],
            },
            'schedule': {
                'hours': 1,
                'range': {
                    'start': '8:00am',
                    'end': '5:00pm',
                },
            },
        },
    })

Likewise, it is possible to set an outage window in which a job is not allowed
to run. The format is the same, by setting the ``invert`` option to ``True``:

.. code-block:: python

    client.api.schedule.add({
        'testjob': {
            'cmd':'local',
            'fun': 'file.touch',
            'tgt': {
                'master1': {
                    'tgt': 'flay',
                    'tgt_type': 'glob',
                },
            },
            'arg': {
                'arg': ['/tmp/someotherfile'],
            },
            'schedule': {
                'hours': 1,
                'range': {
                    'invert': True,
                    'start': '5:00pm',
                    'end': '8:00am',
                },
            },
        },
    })
