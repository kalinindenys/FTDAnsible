# FTD Ansible Modules

A collection of Ansible modules that automate configuration management 
and execution of operational tasks on Cisco Firepower Threat Defense (FTD) devices.

_This file describes development and testing aspects. In case you are looking for 
a client-facing documentation, please check [docsite](./docs/docsite) folder._

## Installation Guide

The project contains Ansible modules for managing device configuration ([`ftd_configuration.py`](./library/ftd_configuration.py)), 
uploading ([`ftd_file_upload.py`](./library/ftd_file_upload.py)) and downloading
([`ftd_file_download.py`](./library/ftd_file_download.py)) files. Sample playbooks are located in 
the [`samples`](./samples) folder.

### Running playbooks in Docker

1. Build the Docker image:
```
docker build -t ftd-ansible .
```

1. Create an inventory file that tells Ansible what devices to run the tasks on. [`sample_hosts`](./inventory/sample_hosts) 
shows an example of inventory file.

1. Run the playbook in Docker mounting playbook folder to `/ftd-ansible/playbooks` and inventory file to `/etc/ansible/hosts`:
```
docker run -v $(pwd)/samples:/ftd-ansible/playbooks -v $(pwd)/inventory/sample_hosts:/etc/ansible/hosts ftd-ansible network_object.yml
```

### Running playbooks locally 

1. Create a virtual environment and activate it:
```
python3 -m venv venv
. venv/bin/activate
```

1. Install dependencies:
`pip install -r requirements.txt`

1. Update Python path to include the project's directory:
```
export PYTHONPATH=.:$PYTHONPATH
```
   
1. Run the playbook:
``` 
ansible-playbook samples/network_object.yml
```

## Unit Tests

The project contains unit tests for Ansible modules, HTTP API plugin and util files. They can be found 
in `test/unit` directory. Ansible has many utils for mocking and running tests, so unit tests
in this project also rely on them and including Ansible test module to the Python path is required.

### Running unit tests in Docker

1. Build the Docker image: 
```
docker build -t ftd-ansible-test test
```
By default, the Dockerfile clones `devel` branch of Ansible repository, but you can change it by adding
`--build-arg ANSIBLE_BRANCH=$BRANCH_NAME` to the build command.

1. Run unit tests with: 
```
docker run -v $(pwd):/ftd-ansible ftd-ansible-test pytest test/
```
To run a single test, specify the filename at the end of command:
```
docker run -v $(pwd):/ftd-ansible ftd-ansible-test pytest test/unit/test_fdm_configuration.py
```

#### Troubleshooting

```
import file mismatch:
imported module 'test.unit.module_utils.test_common' has this __file__ attribute: ...
which is not the same as the test file we want to collect:
  /ftd-ansible/test/unit/module_utils/test_common.py
HINT: remove __pycache__ / .pyc files and/or use a unique basename for your test file modules
```

In case you experience the following error while running the tests in Docker, remove compiled bytecode files files with 
`find . -name "*.pyc" -type f -delete` command and try again.

### Running unit tests locally

1. Clone [Ansible repository](https://github.com/ansible/ansible) from GitHub;
1. Install Ansible and test dependencies:
```
pip install $ANSIBLE_DIR/requirements.txt
pip install test/requirements.txt
```
1. Add Ansible modules to the Python path:
```
export PYTHONPATH=$PYTHONPATH:$ANSIBLE_DIR/lib:$ANSIBLE_DIR/test
```
1. Run unit tests:
```
pytest test/unit
```
    
## Integration Tests

Integration tests are written in a form of playbooks and usually started with `ansible-test` command 
from Ansible repository. As this project is created outside Ansible, it does not have utils to run 
the tests. Thus, integration tests are written as sample playbooks with assertion and can be found 
in the `samples` folder. They start with `test_` prefix and can be run as usual playbooks.

## Debugging

1. Add `log_path` with path to log file in `ansible.cfg`
2. Run `ansible-playbook` with `-vvvv`
    ```
    $ ansible-playbook samples/network_object.yml -vvvv
    ```
3. The log file will contain additional information(REST etc.)
