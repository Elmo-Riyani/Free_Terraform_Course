# What is Infrastructure-as-Code?
Infrastructure-as-Code or IaC for short is just that - it's infrastructure created by code! 

So, instead of manually configuring and deploying infrastructure and its components (like servers, ram, storage, operating systems, etc.), we can define and describe our desired configuration via code, even spinning up 1000s of servers at once! This has a world of benefits. Here's some to think about:
- Infrastructure can now **reliably** be deployed and managed in a **consistent** manner - no more snowflakes! 
- The deployment can be made **automated**.
- It enables **scalability** and **flexibility** - if you need 10 servers instead of 1, just change the number in the code and redeploy!
- It enables **reusability**. Need the same infrastructure deployed in multiple environments (e.g., Prod and Dev)? Easy! 
- Need to provide evidence of configurations and controls for an **audit**? Just share the code!
- IaC is great for **security** as well because tools exist to automatically audit and enforce configurations and controls!

With IaC your infrastructure is essentially software, where it can be put into version-control, tested, and deployed automatically. 
