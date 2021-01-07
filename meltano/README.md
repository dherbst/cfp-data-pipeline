# Meltano - an introduction
I'm not a meltano expert, this is my first time looking at meltano and these types of ELT tools.  The last time I did ELT I wrote my own tools using python, and before that I used Microsoft SSIS pretty extensively.

From meltano.com, meltano is a an open source platform for building, running, and orchestrating ELT pipelines.  These are made up of

- Singer taps (sources) and targets (destinations) these are the EL parts of ELT
- dbt models for transformation - this is the T part of ELT
- airflow orchestration - for creating jobs to run pipelines

The easiest way to get started is to use meltano as a docker image.  You can set up an alias like:

    alias meltano="docker run -it --rm -p 5000:5000 -v ${PWD}:/usr/local/share/cfp -w /usr/local/share/cfp meltano/meltano:latest-python3.8 $@"

And then type a command like `meltano --help`

```
Usage: meltano [OPTIONS] COMMAND [ARGS]...

  Get help at https://www.meltano.com/docs/command-line-interface.html

Options:
  --log-level [debug|info|warning|error|critical]
  -v, --verbose
  --version                       Show the version and exit.
  --help                          Show this message and exit.

Commands:
  add       Add a plugin to your project.
  config
  discover
  elt       meltano elt EXTRACTOR_NAME LOADER_NAME extractor_name: Which...
  init      Creates a new Meltano project
  install   Installs all the dependencies of your project based on the...
  invoke
  schedule
  schema
  select
  ui
  upgrade
  user
```

If you create a project, like `meltano init csv-project` it creates a directory structure for the different ELT tools to communicate in a standard way.

```
csv-project $ tree
.
├── README.md
├── analyze
│   ├── dashboards
│   └── reports
├── extract
│   ├── GitFlixEpisodes.csv
│   ├── GitFlixStreams.csv
│   ├── GitFlixUsers.csv
│   └── files-def.json
├── load
├── meltano.yml
├── model
├── notebook
├── orchestrate
├── requirements.txt
└── transform
```

If you run the `meltano` command without any arguments, it is like running `meltano ui` which starts a webserver on port 5000 which allows you to configure the `meltano.yml` file and see your pipelines and some logs.


# Meltano demonstration project
This runs the csv import tutorial at http://meltano.com/tutorials/csv-with-postgres.html#motivation-and-running-example

This directory already has the gitflix csvs in the `csv-project/extract` directory.   I already ran `meltano init csv-project`.

You can either build your own docker image to install meltano, or you can run their docker image.

To build your own docker image:

    docker build -t melty .

Then run it:

    docker run -it --rm -v $PWD:/usr/local/share/meltano -w /usr/local/share/meltano bash


## Using the meltano image
You can use their image with python3.8 like this, port 5000 is where the UI listens.

    docker run -it --rm -p 5000:5000 -v $PWD:/usr/local/share/cfp/meltano -w /usr/local/share/cfp/meltano --entrypoint bash meltano/meltano:latest-python3.8

Once you are in there at the command line you can run `meltano` commands as you expect to.

I was able to create a csv extract which created these tables:

```
melty-cheese=# \d
         List of relations
 Schema |   Name   | Type  | Owner
--------+----------+-------+-------
 public | episodes | table | admin
 public | streams  | table | admin
 public | users    | table | admin
(3 rows)

melty-cheese=# select * from episodes;
 id | no  |        title        |  tv_series   | rating |  ad_rev   | _sdc_received_at | _sdc_sequence | _sdc_table_version |       _sdc_batched_at
----+-----+---------------------+--------------+--------+-----------+------------------+---------------+--------------------+-----------------------------
 1  | 101 | Pilot               | Breaking Bad | 8.9    | $2,438.13 |                  |    1609955641 |                    | 2021-01-06 17:54:01.9971+00
 2  | 102 | Cat in the Bag...   | Breaking Bad | 8.7    | $1,718.42 |                  |    1609955641 |                    | 2021-01-06 17:54:01.9971+00
 3  | 202 | Grilled             | Breaking Bad | 9.2    | $1,946.21 |                  |    1609955641 |                    | 2021-01-06 17:54:01.9971+00
 4  | 101 | The National Anthem | Black Mirror | 7.9    | $1,198.24 |                  |    1609955641 |                    | 2021-01-06 17:54:01.9971+00
 5  | 406 | Black Museum        | Black Mirror | 8.7    | $1,256.89 |                  |    1609955641 |                    | 2021-01-06 17:54:01.9971+00
 6  | 104 | Old Cases           | The Wire     | 8.3    | $834.67   |                  |    1609955641 |                    | 2021-01-06 17:54:01.9971+00
 7  | 306 | Homecoming          | The Wire     | 8.9    | $764.37   |                  |    1609955641 |                    | 2021-01-06 17:54:01.9971+00
(7 rows)

melty-cheese=# select * from users;
 id |  name  | age | gender |  clv  | avg_logins | logins | _sdc_received_at | _sdc_sequence | _sdc_table_version |       _sdc_batched_at
----+--------+-----+--------+-------+------------+--------+------------------+---------------+--------------------+-----------------------------
 1  | John   | 23  | male   | 163.7 | 0.560009   | 123    |                  |    1609955641 |                    | 2021-01-06 17:54:01.6349+00
 2  | George | 42  | male   | 287.3 | 1.232155   | 147    |                  |    1609955641 |                    | 2021-01-06 17:54:01.6349+00
 3  | Mary   | 19  | female | 150.3 | #DIV/0!    | 0      |                  |    1609955641 |                    | 2021-01-06 17:54:01.6349+00
 4  | Kate   | 52  | female | 190.1 | 0.854654   | 156    |                  |    1609955641 |                    | 2021-01-06 17:54:01.6349+00
 5  | Bill   | 35  | male   | 350.8 | 1.787454   | 205    |                  |    1609955641 |                    | 2021-01-06 17:54:01.6349+00
 6  | Fiona  | 63  | female | 278.5 | #DIV/0!    | 0      |                  |    1609955641 |                    | 2021-01-06 17:54:01.6349+00
(6 rows)

melty-cheese=# select * from streams;
 id | user_id | episode_id | minutes | day | month | year | _sdc_received_at | _sdc_sequence | _sdc_table_version |       _sdc_batched_at
----+---------+------------+---------+-----+-------+------+------------------+---------------+--------------------+-----------------------------
 1  | 1       | 1          | 40      | 10  | 1     | 2019 |                  |    1609955642 |                    | 2021-01-06 17:54:02.2079+00
 10 | 3       | 5          | 41      | 11  | 1     | 2019 |                  |    1609955642 |                    | 2021-01-06 17:54:02.2079+00
 11 | 3       | 1          | 11      | 11  | 1     | 2019 |                  |    1609955642 |                    | 2021-01-06 17:54:02.2079+00
 12 | 4       | 3          | 22      | 10  | 1     | 2019 |                  |    1609955642 |                    | 2021-01-06 17:54:02.2079+00
 13 | 4       | 3          | 18      | 11  | 1     | 2019 |                  |    1609955642 |                    | 2021-01-06 17:54:02.2079+00
 14 | 4       | 6          | 40      | 11  | 1     | 2019 |                  |    1609955642 |                    | 2021-01-06 17:54:02.2079+00
 15 | 5       | 2          | 34      | 11  | 1     | 2019 |                  |    1609955642 |                    | 2021-01-06 17:54:02.2079+00
 16 | 5       | 4          | 41      | 11  | 1     | 2019 |                  |    1609955642 |                    | 2021-01-06 17:54:02.2079+00
 17 | 5       | 5          | 39      | 12  | 1     | 2019 |                  |    1609955642 |                    | 2021-01-06 17:54:02.2079+00
 18 | 5       | 6          | 36      | 12  | 1     | 2019 |                  |    1609955642 |                    | 2021-01-06 17:54:02.2079+00
 19 | 6       | 1          | 19      | 11  | 1     | 2019 |                  |    1609955642 |                    | 2021-01-06 17:54:02.2079+00
 2  | 1       | 2          | 42      | 10  | 1     | 2019 |                  |    1609955642 |                    | 2021-01-06 17:54:02.2079+00
 20 | 6       | 3          | 35      | 11  | 1     | 2019 |                  |    1609955642 |                    | 2021-01-06 17:54:02.2079+00
 21 | 6       | 7          | 48      | 11  | 1     | 2019 |                  |    1609955642 |                    | 2021-01-06 17:54:02.2079+00
 22 | 6       | 1          | 24      | 12  | 1     | 2019 |                  |    1609955642 |                    | 2021-01-06 17:54:02.2079+00
 3  | 1       | 3          | 38      | 11  | 1     | 2019 |                  |    1609955642 |                    | 2021-01-06 17:54:02.2079+00
 4  | 1       | 4          | 12      | 11  | 1     | 2019 |                  |    1609955642 |                    | 2021-01-06 17:54:02.2079+00
 5  | 1       | 5          | 27      | 11  | 1     | 2019 |                  |    1609955642 |                    | 2021-01-06 17:54:02.2079+00
 6  | 2       | 2          | 36      | 11  | 1     | 2019 |                  |    1609955642 |                    | 2021-01-06 17:54:02.2079+00
 7  | 2       | 6          | 45      | 11  | 1     | 2019 |                  |    1609955642 |                    | 2021-01-06 17:54:02.2079+00
 8  | 2       | 7          | 44      | 11  | 1     | 2019 |                  |    1609955642 |                    | 2021-01-06 17:54:02.2079+00
 9  | 3       | 4          | 40      | 10  | 1     | 2019 |                  |    1609955642 |                    | 2021-01-06 17:54:02.2079+00
(22 rows)
```
