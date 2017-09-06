---
title: Walkthrough - Micro Corruption: Cusco
---

## Micro Corruption: Cusco
### Introduction

I have to say up front, this challenge had me stumped for a long, long time. I reckon I spent about 6 hours scrutinizing the program instructions (including drawing out the stack by hand) until I finally figured out where the vulnerability lay. From there, the exploitation was fairly simple. So, if you're struggling with this challenge, I'll offer you a **HINT** so you can keep working at it without any giveaways: **follow the program execution to the very end**. In other words, right up until termination. You've got this!

![New revision, new opportunity](https://mycognosist.github.io/images/CuscoUpdate.png)
_New revision, new opportunity_

As always, the LOCKITALL team have been working to revise their software and make their application more secure. In this version, they've removed the conditional flag we were able to exploit in the Hanoi challenge. That doesn't scare us. Off to work we go!

### The Challenge Begins

Based on the previous challenge, we can begin with the assumption that the program may still be susceptible to a buffer overflow. To test this, we can enter the following password 'AAAABBBBCCCCDDDDEEEEFFFFGGGGHHHHIIIIJJJJKKKKLLLLMMMMNNNNOOOO'. After entering a long password we find that several segments of memory are written to, including part or all of 0x43e0, 0x43f0, 0x4400 and 0x4410.

![Look at that delicious overflow](https://mycognosist.github.io/images/CuscoOverflow.png)
_Look at that delicious overflow_

As in previous challenges, we set about analysing the **test_password_valid** function. However, after extensive analysis we are unable to find an entry point into the overwritten segments of memory that would allow us to potentially redirect the program flow. Where on Earth is the vulnerability?! We may have missed something so we check again. Nope, still nothing. What next?

![Hand-drawn stack interactions](https://mycognosist.github.io/images/CuscoWorking.png)
_My efforts to uncover a vuln in **test_password_valid**. Yup, I even drew out the stack interactions..._

OK, so if the vulnerability isn't in the **test_password_valid** function then it must be after the return the return to the **login** function. Looking at the main function, we see that the output r15 value from **test_password_valid** is tested at address 0x4524 before a jump condition is performed. If the value is not equal to zero, we call the **unlock_door** function. Since we were unable to alter the flow of **test_password_valid**, the return value of r15 is zero and we perform a jump to 0x4532 - resulting in "That password is not correct." being moved into r15. The next instruction performs a call to **puts** before returning to 0x453a. At this point, something interesting happens: 0x10 is added to the stack pointer before the execution of a return function. We set a breakpoint at this instruction (address 0x453a), reset the program and enter our length password. When we arrive at the add #0x10, sp instruction we step through and see that the stack pointer has moved to an address in memory containing part of our password.

![The stack pointer has been redirected to a promising location](https://mycognosist.github.io/images/CuscoSPOver.png)
_The stack pointer has been redirected to a promising location_

The stack pointer is now at a segment of the 0x43f0 memory address. In fact, we can see that it's set to a location with the value 45, the first 'E' in our password. When we step through the ret instruction at 0x453e we notice the program counter (pc) has been set to 4545 - the segment of our password stored at the stack pointer. If we step once more we receive a message at the console: "insn address unaligned". This sets alarm bells ringing in our awareness. We're getting close!

![Address unaligned message](https://mycognosist.github.io/images/CuscoPC.png)
_Address unaligned message. Notice the value of the program counter (pc) in the Register State window?_

Now that we know the program counter is being set based on a value we can control in memory, let's determine the address we'd like to redirect to. A quick look through the **login** function and we see that the **unlock_door** function is called at 0x4528. We lookup the corresponding ascii values of 45 and 28 and discover they translate to 'E' and '(' respectively. Great, now let's reformulate our password to include these values (remembering to reverse their order due to Endianess) and rerun the program. We step through the ret instruction at 0x453e after the add #0x10, sp instruction and we arrive just where we had hoped: 0x4528 (the call for **unlock_door**).

![Successful redirect due to stack overflow](https://mycognosist.github.io/images/CuscoPass.png)
_Successful redirect due to stack overflow_

Continue through the call instruction...cha-ching - jackpot!

![Yes, what a feeling!](https://mycognosist.github.io/images/CuscoSolved.png)
_Yes, what a feeling!_

### Conclusion

The main lesson I gained through this challenge was to not make assumptions about where a program terminates, or where a potential vulnerability lies. Don't give up until the very last instruction has been completed! While I may have spent hours searching for a vulnerability in the wrong location, all that effort helped me gain a much better understanding of the stack and how values of registers are temporarily stored in memory. Time well spent!

I'd love to receive feedback on these write-ups so you're invited to follow me on Twitter [@mycognosist](https://twitter.com/mycognosist) and let me know what you think. I'm always happy to connect with fellow geeks so feel free to reach out :) Thanks for reading and happy reverse-engineering!

[Back to the walkthrough index](https://mycognosist.github.io/">Back to walk-through index)
