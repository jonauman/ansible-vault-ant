# Ant properties encryption with ansible-vault

This demo code shows how you can encrypt variables using ansible vault, and then make those variables accessible to ant tasks as properties.

# Setup
- encrypt variables in secrets.yml

  ```
  ansible-vault create secrets.yml
  ```

  The contents of this file will be:

  ```
  ---
  test_password: rumpelstiltskin
  ```

- include secrets.yml in your ansible task
  ```
  # vault.yml
  tasks:
    - include_vars: secrets.yml
  ```
- run ant in your ansible playbook using the 'environment' keyword
  ```
    - name: run ant
    shell: ant -f vault.xml chdir='ant/'
    environment:
      test_password: test_password
  ```
  Note: Here I am creating an environment variable 'test_password' with the value for test_password found in secrets.yml
  
- In my ant build file, I will access the environment vairable like this:
  ```
  <property environment="env"/>
  <target name="main">
    <echo message="[ANT TASK]: test_password is ${env.test_password}" />
  </target>
  ```
  
# Execute playbook
```
ansible-playbook -vvv -i inventory  --vault-password-file=.vault.pwd vault.yml
```
Running ansible in verbose mode will show you that ant is now able to access the environment variable as a property. The variable is only available in memory while ansible is running. The user running ansible does not have access to the environment variable outside of running the ansible playbook. Neither does ant have access to the property outside of the ansible task.

# Sample ansible output
```
TASK: [run ant] ***************************************************************
<localhost> REMOTE_MODULE command ant -f vault.xml chdir='ant/' #USE_SHELL
...
changed: [localhost] => 
{
    "changed": true,
    "cmd": "ant -f vault.xml",
    "delta": "0:00:00.454655",
    "end": "2015           -10-08 09:41:33.569236",
    "rc": 0,
    "start": "2015-10-08 09:41:33.114581",
    "stderr": "",
    "stdout": "Buildfile:            vault.xml\n\nmain:\n     [echo] [ANT TASK]: test_password is rumpelstiltskin\n\nBUILD SUCCESSFUL\nTotal tim           e: 0 seconds",
    "warnings": []
}
```

# Run as jenkins job
- Checkout this repo
- Execute run.sh in your job
- For obvious security reasons, you should not include the password file in your repo. Instead, the jenkins administrator should reference a password file outside of the repo and only accessible by the jenkins user. Ansible vault also accepts a script to output the password to stdout.
