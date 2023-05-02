Original post:

<https://www.cyberciti.biz/faq/how-to-create-tar-gz-file-in-linux-using-command-line/>

---

## Introduction:&#x20;

A tarball (or tar.gz or tar.bz2) is nothing but a computer file format that combines and compresses multiple files. Tarballs are common file format on a Linux or Unix-like operating systems. Tarballs are often used for distribution of software/media or backup purposes. This page shows how to create tar.gz file in Linux using the Linux command line.

+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------------------------------------------------------------------------------------------+
|**Tutorial requirements**                                                                                                                                                                                  |<br>                                                                                                     |
+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------------------------------------------------------------------------------------------+
|Requirements                                                                                                                                                                                               |Linux                                                                                                    |
+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------------------------------------------------------------------------------------------+
|Root privileges                                                                                                                                                                                            |No                                                                                                       |
+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------------------------------------------------------------------------------------------+
|Difficulty level                                                                                                                                                                                           |[Easy](http://www.cyberciti.biz/faq/tag/easy/ "See all Easy Linux / Unix System Administrator Tutorials")|
+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------------------------------------------------------------------------------------------+
|Est. reading time                                                                                                                                                                                          |5m                                                                                                       |
+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------------------------------------------------------------------------------------------+
|**Table of contents ↓**                                                                                                                                                                                    |<br>                                                                                                     |
|                                                                                                                                                                                                           |                                                                                                         |
|- [1 tar command](https://www.cyberciti.biz/faq/how-to-create-tar-gz-file-in-linux-using-command-line/#tar_command "tar command")                                                                          |                                                                                                         |
|- [2 Creating tar.gz file](https://www.cyberciti.biz/faq/how-to-create-tar-gz-file-in-linux-using-command-line/#Creating_tar.gz_file "Creating tar.gz file")                                               |                                                                                                         |
|- [3 How to create tar gz file (Example)](https://www.cyberciti.biz/faq/how-to-create-tar-gz-file-in-linux-using-command-line/#How_to_create_tar_gz_file_\(Example\) "How to create tar gz file (Example)")|                                                                                                         |
|- [4 Understanding tar command options](https://www.cyberciti.biz/faq/how-to-create-tar-gz-file-in-linux-using-command-line/#Understanding_tar_command_options "Understanding tar command options")        |                                                                                                         |
|- [5 Conclusion](https://www.cyberciti.biz/faq/how-to-create-tar-gz-file-in-linux-using-command-line/#Conclusion "Conclusion")                                                                             |                                                                                                         |
+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------------------------------------------------------------------------------------------+

## Say hello tar command

You need to use the tar command. The tar command first offered in the seventh edition of unix v7 in January 1979. Each tarball stores data along with additional information such as:

- File names
- Directory structure
- File permission
- SELinux permissions
- Timestamps
- File ownership
- File access permissions and more

## How to create tar.gz file in Linux using command line

The procedure to create a tar.gz file on Linux is as follows:

1. Open the terminal application in Linux
2. Run tar command to create an archived named `file.tar.gz` for given directory name by running: `tar -czvf file.tar.gz directory`
3. Verify `tar.gz` file using the ls command and tar command

Let us see all commands and options in details.

## Linux create tar.gz file with tar command

Say you want to create tar.gz file named projects.tar.gz for directory called $HOME/projects/:\
`$ ls -l $HOME/projects/`\
Sample outputs:

```shell
# sample output

total 8
drwxr-xr-x 2 vivek vivek 4096 Oct 30 22:30 helloworld
drwxr-xr-x 4 vivek vivek 4096 Oct 30 13:16 myhellowebapp
```

The syntax for the tar command is as follows:

```shell
tar -czvf filename.tar.gz /path/to/dir1
tar -czvf filename.tar.gz /path/to/dir1 dir2 file1 file2

# Create a tar.gz file from all pdf (".pdf") files
tar -czvf archive.tgz *.pdf
```

\
To create projects.tar.gz in the current working directory, run:\
`$ tar -czvf projects.tar.gz $HOME/projects/`\


### Verification

Next verify newly created tar.gz file in Linux using the ls command:\
`ls -l projects.tar.gz`\
Example outputs:

```shell
-rw-r--r-- 1 vivek vivek 59262 Nov  3 11:08 projects.tar.gz
```

One can [list the contents of a tar or tar.gz file](https://www.cyberciti.biz/faq/list-the-contents-of-a-tar-or-targz-file/) using the tar command:

```shell
tar -ztvf projects.tar.gz
```

<br>

![](https://www.cyberciti.biz/media/new/faq/2013/06/Linuxlist-tar.gz-file-contents-300x145.png)

## How to extract a tar.gz compressed archive on Linux

The syntax is follows to extract tar.gz file on Linux operating system:

```shell
tar -xvf file.tar.gz
tar -xzvf file.tar.gz
tar -xzvf projects.tar.gz
```

One can extract files in a given directory such as `/tmp/:$ tar -xzvf projects.tar.gz -C /tmp/`

## Sample outputs:

```shell
home/vivek/projects/myhellowebapp/
home/vivek/projects/myhellowebapp/hellowebapp.working/
home/vivek/projects/myhellowebapp/hellowebapp.working/db.sqlite3
home/vivek/projects/myhellowebapp/hellowebapp.working/manage.py
home/vivek/projects/myhellowebapp/hellowebapp.working/collection/
home/vivek/projects/myhellowebapp/hellowebapp.working/collection/models.py
home/vivek/projects/myhellowebapp/hellowebapp.working/collection/admin.py
home/vivek/projects/myhellowebapp/hellowebapp.working/collection/__init__.py
home/vivek/projects/myhellowebapp/hellowebapp.working/collection/tests.py
home/vivek/projects/myhellowebapp/hellowebapp.working/collection/static/
home/vivek/projects/myhellowebapp/hellowebapp.working/collection/static/css/
home/vivek/projects/myhellowebapp/hellowebapp.working/collection/static/css/style.css
...
..
...
home/vivek/projects/myhellowebapp/hellowebapp/hellowebapp/__pycache__/urls.cpython-36.pyc
home/vivek/projects/myhellowebapp/hellowebapp/hellowebapp/wsgi.py
home/vivek/projects/myhellowebapp/hellowebapp/hellowebapp/settings.py
home/vivek/projects/helloworld/
```

### Understanding tar command options

| tar command option | Description                                                         |
| ------------------ | ------------------------------------------------------------------- |
| -c                 | Create a new archive                                                |
| -x                 | Extract files from an archive                                       |
| -t                 | List the contents of an archive                                     |
| -v                 | Verbose output                                                      |
| -f file.tar.gz     | Use archive file                                                    |
| -C DIR             | Change to DIR before performing any operations                      |
| -z                 | Filter the archive through gzip i.e. compress or decompress archive |

<br>
