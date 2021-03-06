# Visual Road: A Video Data Management Benchmark

Please see the [project website](https://db.cs.washington.edu/projects/visualroad) for more details about the benchmark, links to the papers, sample videos, and pregenerated datasets.

## Building the Visual Road Docker Image

Because of licensing restrictions on the Unreal engine, we cannot release a pre-built Docker container for the Visual Road benchmark.  However, we have striven to make the build process as painless as possible!  Note that Visual Road depends on Unreal version 4.22.0 and only supports Linux builds.

1. Install [Docker CE](https://docs.docker.com/install/linux/docker-ce/), if not already installed.
2. Install [Python 3.6](https://www.python.org/downloads/) or later, if not already installed:

```sh
sudo apt-get install python3 python3-dev python3-pip
```

3. Install `nvidia-docker` version 1.0.1 or later:

```sh
sudo apt-get install nvidia-docker
```

4. Install `ue4-docker` version 0.0.34 or later:

```sh
sudo pip3 install ue4-docker
sudo ue4-docker setup
``` 

5. You will need an Unreal Engine account in order for `ue4-docker` to be able to download the engine.  If you don't already have an account, [create one](https://accounts.unrealengine.com/login).

6. Build the `ue4-engine:4.22.0` image using `ue4-docker` and CUDA version 9.2.  You will be prompted for your Unreal Engine account information from the previous step in order to build the engine:

```sh
ue4-docker build --cuda=9.2 --pull-prerequisites --no-minimal --no-full --exclude=debug --exclude=templates 4.22.0
```

7. Clone the [Visual Road repository](https://github.com/uwdb/visualroad) and build the benchmark image:

```sh
git clone https://github.com/uwdb/visualroad
cd visualroad
docker build -t visualroad/core .
```

## Synthetic Dataset Generation

1. Initiate dataset generation by running the `generator` service.  For example, the following command generates a scale-one dataset named `my-dataset`: `docker-compose run generator --scale 1 my-dataset`.
2. The generator service supports a number of additional options (e.g., `--height`, `--width`, `--duration`).  Execute `docker-compose run generator -h` for a complete list.

## Generating Benchmark Queries

1. After creating a synthetic dataset, generate a suite of benchmark queries by invoking the driver service: `docker-compose run driver [path to synthetic dataset]`.  The dataset path must be _a subdirectory within the current directory_ (e.g., `/home/root/dataset` will not work).
2. The driver will emit YAML to standard output with query parameters.  As an example, an abbreviated output for the first two instances of Q1 might read:

```yml
source: my-dataset
batches:
- batch:
  query: '1'
  - query:
      path: traffic-000.mp4
      t:
      - 2
      - 30
      x:
      - 360
      - 1875
      y:
      - 683
      - 1001
  - query:
      path: traffic-002.mp4
      t:
      - 18
      - 228
      x:
      - 1481
      - 1736
      y:
      - 825
      - 902
...
```

## Verifying Query Results

1. Visual Road's verification service may be optionally invoked to ensure that query results conform with the benchmark requirements.  To do so, execute the verification service: `docker-compose run verifier -q [path to driver query YAML] -r [result YAML]`.  The driver YAML is the file generated by the previous step, and the result YAML is a user-constructed file using the following format:

```yml
- result:
  query: '1'
  - result-q1-1.mp4 # Path to result for instance 1 of query 1
  - result-q1-2.mp4 # Paths must be under the directory where the verifier is invoked
  - result-q1-3.mp4
  - result-q1-4.mp4

- result:
  query: '2a'
  - result-q2a-1.mp4
  - result-q2a-1.mp4
  - result-q2a-1.mp4
  - result-q2a-1.mp4

...  
```

2. The verifier may be applied to datasets other than the one on which the driver was executed.  To do so, execute the verifier with the `--dataset` option.  See the verifier help (`-h`) for other options.

## Share your configuration!

If you've generated a dataset and would like to share its configuration with the world, please [post it here](https://github.com/uwdb/visualroad/issues/new?labels=Benchmark+Configuration&template=benchmark-configuration.md) with the details!  To view a list of existing dataset configurations, please [click here](https://github.com/uwdb/visualroad/issues?q=label%3A%22Benchmark+Configuration%22).

## Citations & Paper

If you use Visual Road, please cite our SIGMOD'19 paper:

_Visual Road: A Video Data Management Benchmark_<br />
Brandon Haynes, Amrita Mazumdar, Magdalena Balazinska, Luis Ceze, Alvin Cheung<br />
SIGMOD:972-987 [[PDF]](https://dl.acm.org/citation.cfm?doid=3299869.3324955)

```
@inproceedings{DBLP:conf/sigmod/HaynesMBCC19,
  author    = {Brandon Haynes and Amrita Mazumdar and Magdalena Balazinska and Luis Ceze and Alvin Cheung},
  title     = {Visual Road: {A} Video Data Management Benchmark},
  booktitle = {{SIGMOD}},
  pages     = {972--987},
  year      = {2019},
  doi       = {10.1145/3299869.3324955},
}
```

See the [project website](http://visualroad.uwdb.io) for more details about the benchmark and related papers.
