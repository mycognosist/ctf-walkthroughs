---
title: Walkthrough - Micro Corruption: Hanoi
---

## Micro Corruption: Hanoi
### Introduction

In my previous two Micro Corruption walkthroughs I went into quite a lot of detail, mostly because I was writing for an audience who is brand new to reverse-engineering (like myself). From this walkthrough onwards I'll aim to be a little less verbose and focus on the mechanics of solving the challenge. With that said, let's jump into the action!

### The Challenge Begins

The structure of the program and its flow are once again very similar to the previous challenge. The **login** function asks for a password as input before calling the **test_password_valid** function. Upon returning to the **login** function, the value of r15 is tested and the result determines the outcome of a jump condition. If the value of r15 is zero, we jump to unlock the door. If not zero, we move down to the next instruction: 0xdd is moved into memory at 0x2410, "Testing if password is valid" is printed to the screen, and we then meet a compare instruction at 455a. In the compare instruction, 0x45 is compared with the value stored in memory at 0x2410. If the values are equal we step down to the next instruction, resulting in "Acess granted" and an unlocked door. If not equal, we jump to address 4570 and fail in our quest to unlock the device. Hmm...interesting.

If we focus our analysis on the **test_password_valid** function, we quickly discover that it's not possible to return to **login** with a r15 value of zero - no matter what password we try. However, the value of r15 isn't the only route to unlock the device. As mentioned in the previous paragraph, a second conditional flag exists at address 455a where the outcome of a compare instruction can result in the device being unlocked. This looks to be our best opportunity for exploitation.

In order to exploit this potential vulnerability, we'd need to overwrite the value of 0x2410 in memory. At this point, if you're unfamiliar with buffer overflows, I suggest you check out [LiveOverflow's amazing tutorial series](https://www.youtube.com/playlist?list=PLhixgUqwRTjxglIswKp9mpkfPNfHkzyeN) before continuing. We know from our analysis of the **login** function that the password we enter is stored at 0x2400 in memory. How do we know this? Well, the value of 0x2400 is moved into r15 just before the **test_password_valid** is called. We can also see the hex values of our password stored in 0x2400 when we enter 'password' as our password.

![Notice 'password' stored at 0x2400?](https://mycognosist.github.io/images/HanPass2400.png)
_Notice 'password' stored at 0x2400?_

Let's see if we can overwrite the contents of 0x2410 by overflowing the password buffer. We enter a string like 'AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA' as our password and check the memory.

![Our password overflows into 0x2410](https://mycognosist.github.io/images/HanAOFlow.png)
_Our password overflows into 0x2410_

Bingo! We've successfully overwritten the value of 0x2410 with '41' - the hex value of 'A'. However, if we want the conditional flag (compare at address 445a) to lead to unlocking the door, we need the value at 0x2410 to equal '45'. A quick check with a hex to ascii converter reveals '45' to be 'E' in ascii. Since the 17th character of our password begins the overflow into 0x2410, we can try 'AAAAAAAAAAAAAAAAE' as our password.

![Looking good: value of '45' at 0x2410](https://mycognosist.github.io/images/HanEOFlow.png)
_Looking good: value of '45' at 0x2410_

Now we just need to step through the program and...

![Ding ding ding, we have a winner!](https://mycognosist.github.io/images/HanUnlocked.png)
_Ding ding ding, we have a winner!_

We did it! We successfully executed a buffer overflow to direct a conditional flag outcome towards unlocking the door. Great work! Now simple 'solve' the challenge with the successful password. As a quick note, you may find it helpful to enter characters in sequences of 4 when attempting a buffer overflow (e.g. 'AAAABBBBCCCCDDDDEEEE'). This will help you to determine where the overflow is occuring and at what point in your password string to insert desired values.

### Conclusion

Well, we completed our first buffer overflow of the Micro Corruption challenges...sweet! The knowledge and experience gained in this challenge will certainly be necessary as we move forward through the challenges. A big thanks to [LiveOverflow](https://twitter.com/LiveOverflow) for his fantastic [Binary Hacking tutorial series](https://www.youtube.com/playlist?list=PLhixgUqwRTjxglIswKp9mpkfPNfHkzyeN)! The dude is a legend and his explanations have helped me a lot in understanding instruction flow and stack dynamics. If you're ever stuck with anything related to reverse-engineering, be sure to check out his video. Until next time, thanks for reading and happy reverse-engineering!

[Back to the walkthrough index](https://mycognosist.github.io/)
