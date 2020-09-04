For the Docker image to build properly with GeneMarkS, you need to download it from the GeneMark website (http://topaz.gatech.edu/GeneMark/license_download.cgi).

Download "GeneMarkS v.4.30" for "LINUX 64".

After registering, you should be given the opportunity to download files called "genemark_suite_linux_64.tar.gz" and "gm_key_64.gz".

Store those in this directory, and then run the `build_docker` script to build the image.

Then execute the `run_docker` script to start the container for the back end.

It should run on port 5000.
