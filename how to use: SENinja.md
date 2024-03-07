1. install SEninja:\
![image](https://github.com/Boberttt/Reverse-Engineering/assets/104478197/bca49d5e-9fd1-4e12-aaca-43564ad4c408)

2. open your binary, I will be using a file named a.exe, which is in this github.
3. Go to the SENinja window:\ ![image](https://github.com/Boberttt/Reverse-Engineering/assets/104478197/b57fbff9-8865-4405-a9db-c56cc163cdb8)
4. Identify where you want to start execution, for me it will be here due to me being able to replace the non-existent buffer with my symbolic buffer:\ ![image](https://github.com/Boberttt/Reverse-Engineering/assets/104478197/a2a61320-f30a-45db-858d-2280d8ef1571)
5. Select your start instruction and press start:\ ![image](https://github.com/Boberttt/Reverse-Engineering/assets/104478197/50c15ade-1c14-4bd5-b8ce-01f52d2fac81)
7. Identify your target instruction, in this case mine will be:\ ![image](https://github.com/Boberttt/Reverse-Engineering/assets/104478197/7d2f88f0-8d49-4346-b642-b638287d3f8a)
8. While that instruction is selected, press this:\ ![image](https://github.com/Boberttt/Reverse-Engineering/assets/104478197/62bac00e-4671-433c-9f96-7d879364041d)
9. Identify your instruction to be avoided, select it, press this:\ ![image](https://github.com/Boberttt/Reverse-Engineering/assets/104478197/8f6eb735-1633-40d1-ada0-8c28500fc4be)
10. Now go to buffers and select "Create new buffer":\ ![image](https://github.com/Boberttt/Reverse-Engineering/assets/104478197/f3648c12-f83b-4ab5-bcba-68fc06ad3dfb)
11. Leave the name unchanged and change the value to something either equal to or higher then your known string:\ ![image](https://github.com/Boberttt/Reverse-Engineering/assets/104478197/58e732cb-823c-4722-a4e1-ae16bb6929d5)
12. Go to registers, right click RAX (it may not be RAX for your specific binary), and press bind to symbolic buffer, then press ok:\ ![image](https://github.com/Boberttt/Reverse-Engineering/assets/104478197/fd36ae32-6fc4-4adc-b633-dcaca4deec49)
13. Now run either DFS or BFS, both should work:\ ![image](https://github.com/Boberttt/Reverse-Engineering/assets/104478197/8bc6f462-bee6-4593-a674-cf1bf08954ec)
14. Wait
