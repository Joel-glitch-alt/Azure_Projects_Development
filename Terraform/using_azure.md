1. Generate new keys directly on Desktop:

ssh-keygen -t rsa -b 4096 -f C:\Users\jjDok\Desktop\id_rsa

2. When SSH-ing into your VM later, you'll need to specify the key location:

ssh -i C:\Users\jjDok\Desktop\id_rsa adminuser@<vm-ip-address>