![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/6cnjpie87oe1wxdfy3bc.jpg)
# Ansible Vault Quick Start
- Ansible Vault encrypts variables and files so you can protect sensitive content such as passwords or keys rather than leaving it visible as plaintext in playbooks or roles. 

- 5 steps or less to quick start with ansible vault

### **1. Create password stored file to hold the master key**
```
~: cat passwd_file
pLECtontABre
```
### **2. Generate encrypted password and data conent**
- Password for ssh-login
```
 $ ansible-vault encrypt_string --vault-id vagrant@passwd_file 'rootroot' --name vagrant
vagrant: !vault |
          $ANSIBLE_VAULT;1.2;AES256;vagrant
          33383365353665376338326665343364373334633265633438366263336638393937366661333430
          6662656233333563333461333962376539353634643563630a386335666632656435363863366339
          39623635363930373264636234653632616536343064313134623535653833353737313236313064
          6335323136366438640a373131373764393337303734386631303133336234343638623233356430
          3135
Encryption successful
```

- Data encrypt
```
⚡ $ ansible-vault encrypt_string --vault-id dev@passwd_file 'my_credential_data' --name 'my_data_encrypted' 
my_data_encrypted: !vault |
          $ANSIBLE_VAULT;1.2;AES256;dev
          65653537303030393632656639333262346565383266643734353865356163363538613736316465
          3734303334396561626538386466356461643035373166360a383735343735663234653736626565
          63363436626135373366633034353762623366333664633964666463373765396335373737313035
          3062393261613966360a373533326637323832633330303230383036636435383165376230656133
          31366231326330336137633035623763396533313735636531623438386632376536
Encryption successful
```

- Update the encrypted things to `var.yaml`
```
vagrant: !vault |
          $ANSIBLE_VAULT;1.2;AES256;vagrant
          33383365353665376338326665343364373334633265633438366263336638393937366661333430
          6662656233333563333461333962376539353634643563630a386335666632656435363863366339
          39623635363930373264636234653632616536343064313134623535653833353737313236313064
          6335323136366438640a373131373764393337303734386631303133336234343638623233356430
          3135

my_data_encrypted: !vault |
          $ANSIBLE_VAULT;1.2;AES256;dev
          65653537303030393632656639333262346565383266643734353865356163363538613736316465
          3734303334396561626538386466356461643035373166360a383735343735663234653736626565
          63363436626135373366633034353762623366333664633964666463373765396335373737313035
          3062393261613966360a373533326637323832633330303230383036636435383165376230656133
          31366231326330336137633035623763396533313735636531623438386632376536
```

### **3. Create ansible task to test decryption**
- test.yml: Variable `my_data_encrypted` as encrypted value
```
⚡ $ cat test.yml 
---
- hosts: all
  become: true
  vars_files:
    - var.yaml
  tasks:
    - name: ansible vault test
      debug:
        msg: "{{ my_data_encrypted }}"
```
- inventory: variable `vagrant` as the encrypted value
```
⚡ $ cat inventory 
[ubuntu]
192.168.121.21

[all:vars]
ansible_connection=ssh
ansible_user=vagrant
ansible_ssh_pass={{ vagrant }}
```

### **4. Run ansible-playbook**
```
⚡ $ ansible-playbook test.yml --vault-id dev@passwd_file -i inventory -e @var.yaml
ok: [192.168.121.107]
 ___________________________
< TASK [ansible vault test] >
 ---------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||

ok: [192.168.121.107] => {
    "msg": "my_credential_data"
}
 ____________
< PLAY RECAP >
 ------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||

192.168.121.107            : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
