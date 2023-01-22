# Part 1: Docker & SQL

### Question 1 - knowing docker tags:

`docker --help` tells us we need to run `docker build --help` to find out more about the `docker build` command.

We can see that  `--iidfile string` corresponds to "Write the image ID to the file".


### Question 2 - understanding docker first run:

The following command runs docker with the python:3.9 image in an interactive mode and the entrypoint of bash:

`docker run -it --entrypoint=bash python:3.9`

Running `pip list` in the bash prompt tells us that there are 3 packages installed.


### Question 3 - count records:


