# Accessing and Downloading Cal-Adapt: Analytics Engine data

Today I am going to demo 3 of the many ways to access and download the nearly one petabyte of Cal Adapt Analytics Engine data stored in the cloud. With such a large amount of data it is very useful to have software tools such as Amazon Web Services Command Line Interface, AWS CLI, and Python to aid us in accessing and downloading the data.

## Software Used in this demonstration

### AWS Command Line Interface (AWS CLI)

The [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) can be set up easily for various operating systems. You will just need the software itself and not have to worry about AWS accounts or priviledges as all the CA: AE data is public.

### Python virtual environment

For this workshop we will be using a Python virtual environment. The [CA: AE repo Wiki](https://github.com/cal-adapt/climakitae/wiki) has instructions for setting up a Python virtual environment in either pip or conda. These instructions will get all the dependent packages, as well as climakitae installed.

## Methods for accessing and downloading covered in this workshop:

- [Downloading from AWS S3 using AWS CLI](aws_cli_downloading.md)

- [Accessing data using Intake ESM Python package](intake_downloading.md)

- [Accessing data using climakitae Python package](get_data_downloading.md)
