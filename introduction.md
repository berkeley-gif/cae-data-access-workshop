# Accessing and Downloading Cal-Adapt: Analytics Engine data

This workshop will go over 3 of the many ways to access and download the nearly 1 petabyte of Cal-Adapt: Analytics Engine (CA: AE) data stored in the Cloud. With the scale of the data it is very useful to have software tools, such as AWS CLI and Python, to aid in accessing and downloading the data.

## AWS Command Line Interface (AWS CLI)

The [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) can be set up easily for various operating systems. You will just need the software itself and not have to worry about AWS accounts or priviledges as all the CA: AE data is public.

## Python virtual environment

For this workshop we will be using a Python virtual environment. The [CA: AE repo Wiki](https://github.com/cal-adapt/climakitae/wiki) has instructions for setting up a Python virtual environment in either pip or conda. These instructions will get all the dependent packages, as well as climakitae installed.

## Methods for accessing and downloading covered in this workshop:

- [Downloading from AWS S3 using AWS CLI](aws_cli_downloading.md)

- [Accessing data using Intake ESM Python package](intake_downloading.md)

- [Accessing data using climakitae Python package](get_data_downloading.md)