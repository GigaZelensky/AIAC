# AI-AC: Semi-Open Source Time-Frequency Transform + Deep Learning Cloud Anti-Cheat  
*(Cloud server side is also made public)*

### Abandoned. Goodnight, AIAC.

## **Source Code**

```
https://github.com/huzpsb/AIAClient
```

## **Introduction**

The AI-led anti-cheat market has been monopolized by closed-source, paid solutions for a long time. I want to break that monopoly.

This project is only a Proof of Concept (POC) and doesn’t fully integrate other components beyond the AI itself. In other words, its only function is: **assuming every possible attack or bypass can be completed**, it detects whether a player is using a third-party aiming tool. (It’s recommended to pair this with Grim.)

[![State-of-the-art Shitcode](https://img.shields.io/static/v1?label=State-of-the-art&message=Shitcode&color=7B5804)](https://github.com/trekhleb/state-of-the-art-shitcode)

## **Principle**

Since this tool is purely a technical test, let’s talk about how it works rather than how to use it right away.

---

### Sampling

The sampling component is **fully open source**.

It consists of two steps:

1. **Compute SHB (SignedHitBox)**

   What is SHB? In simple terms, SHB’s size is equal to the smallest square HitBox needed to attack an entity. 

   Still unclear? Don’t worry, here’s a picture:

   ![image](https://user-images.githubusercontent.com/41772578/175223257-36d1891e-bdb7-4eeb-8175-429dc9b9820e.png)

   The *signed* part of SHB is determined by whether the arrow (direction) relative to the connecting line is clockwise or counterclockwise.

2. **FFT**

   What is FFT? *Sigh*, I’m tired of typing. Fine, [here’s a portal](https://zhuanlan.zhihu.com/p/347091298).

   Some might ask, “Wait, SHB is a real number. How does that become a complex number?”  
   Well, a real number is just a special case of a complex number. Think about it carefully.

---

### Communication

The client side of communication is **open source**.

Communication is just a normal C/S architecture. When sampling, we get a stream of data, which makes FFT and feature extraction complicated. How do we deal with that?

We slice it. I choose to slice every group of 22 data points, then send that slice to the server. The server calculates and returns the classification result.

**But wait**, this begs the question: why does an anti-cheat plugin need a C/S architecture in the first place?

~~Because Matrix, Reflex, and AAC all do that, so I directly borrowed the concept (plus hopping on the “cloud” bandwagon).~~  
Actually, it's due to **package size**. As we all know, the biggest drawback of using Java is that it’s slow—way too slow. C++ is faster. But C++ is harder to write and can’t directly interface with Minecraft. 

We have something called JavaCPP as a bridge. But JavaCPP is quite large, so after packaging, it directly bloats to about 150MB.

A 150MB plugin is clearly not feasible, right?

So the only solution is a C/S architecture. The server is multi-threaded, so it won’t lag. I also did a baseline performance test: on my laptop, it can handle 150 requests per second, which should be enough for up to 10,000 players in Bedwars (assuming each player has a valid attack every three seconds).

---

### Training & Computation

This part is **not open source**, but it is **not obfuscated**. You can do as you wish with it.

Even though communication is done, the hardest part remains. Let’s dive in!

I built a 5-layer neural network with node counts 22, 30, 30, 30, and 3:

![image](https://user-images.githubusercontent.com/41772578/175223295-cd1115aa-a402-45a6-8a75-d3069fd63cf5.png)

Initially, I thought all my efforts were in vain because the model just wouldn’t converge. One time, by accident, I increased the training iterations by an extra 100 million (`亿` in Chinese means 100 million), then went to sleep.

![image](https://user-images.githubusercontent.com/41772578/175223325-fe147988-2790-4aa7-8358-dbb3ee9c24e7.png)

The next day, I found that after 130,000 training epochs, the network had a posterior accuracy of 99%.

This model is what I’ve included in the demo release.

As for computation, there’s nothing particularly new, so let’s skip it (doge).

**FIXME**: This section of the documentation is outdated. I currently use adaptive training, and the model has been adjusted following an autoencoder approach. However, this doesn’t affect the general understanding. If anyone is willing to update the docs via PR, I’ll happily ~~procrastinate~~ accept it.

---

## **Usage**

~~Finally we’re getting to it, huh?~~

1. **Run the AI-AC server**.

   ![image](https://user-images.githubusercontent.com/41772578/175223344-6ce7ae22-fe92-4540-8c4f-d060a5f946b2.png)

   You probably know how to do that, right?

2. **Create an account** for yourself.

   ```
   set <username> <valid_days> <ip> <max_call_limit>
   ```

   Here, `username` is just an identifier and not related to the MC plugin.

   In fact, there’s no login configuration in the MC plugin at all.

   Some people might ask, “You’ve written all these features for monetization—can I use them to charge money?”

   Well, my original intention with these features was to allow groups to share anti-cheat computing resources and easily measure each party’s contribution, not for profiteering. But if you can simultaneously comply with the AGPL and still make money, I can’t stop you.

3. **Configure the MC server**

   The server config file is as follows:

   ```yaml
   # Whether to show debug info like sequence exceptions, VL, etc.
   debug: false
   # AI server address
   server: "127.0.0.1"
   # AI server port
   port: 1451
   # Punishment command
   cmd: "ban %p"
   # How much VL to add for each violation
   sec: 5
   # Natural VL decay
   fall: 2
   # Trigger punishment once VL reaches this threshold
   vl: 15
   # Default label/tag
   tag: "unset"
   # Data collection mode: only collects data, no calculations. Good for testing!
   dev: false
   # Stream sampling factor. Must not exceed 10!
   stay: 5
   ```

   Make sure `server` and `port` match the AI server from earlier.

   Then set up your punishment command. That’s it.

   By the way, this plugin is not suitable for production use yet, but for testing it should be fine.

---

## **Data Donation**

**You don’t need to know Java, and it won’t affect gameplay. Your data is extremely valuable to me!**

It’s very simple:

1. Enter `/type <YourDataName>`
2. Continue playing.
3. When you’re done, send me the `AI-AC` folder from your server.

Please **don’t donate garbage data** (for example: deliberately shaking your mouse wildly, attacking entities that have no knockback or have a different size than players, or using blatant cheats in the absence of other AC plugins). It’s not only for the community’s sake, but also for yours ↓

This project is non-profit, so we can’t provide extra perks to donors. But if necessary, we can offer a basic model trained an additional 100 epochs on your custom data (you must donate at least 1,000 data samples, because that’s a minimum of 1,000 × 100 = 100,000 training operations).

“Nobody understands your needs better than you do.” A custom-trained model can give you surprising results!

*(To save computing power, this extra perk isn’t provided by default. If you need it, please join our QQ group and let me know.)*

**Privacy Policy**: The custom model you train won’t be made public; however, your donated data (in anonymized form) will be used for community base model training. We do not keep raw data. Please submit at least 1,000 samples in a single batch. Please do not resubmit the same data multiple times, or it may be flagged as garbage data due to anomalous training metrics.

**Data Name** should include:

- Your nickname
- Indicate whether it’s hand-played (manual) or “G” (cheat).  
  If “G,” specify whether it’s KA Single, KA Multi, or AimBot (Trigger/AutoClicker does not count as G).
- Beyond that, try not to change your data name often. Many thanks!

---

## **Download**

[https://share.weiyun.com/ncJbAQ00](https://share.weiyun.com/ncJbAQ00)

Note: This download package has “everything,” but I might release partial updates to some components in the QQ group.

That means all incremental updates build on this package. If that’s confusing, just join the group and grab the latest version there. =.=

## **The End**

- **To donate data:** Email `huzpsb@qq.com` with the subject `[AI-AC Donation]` or join the QQ group.
- **To donate funds:** [https://afdian.net/@huzpsb](https://afdian.net/@huzpsb)

Thanks to:
- **PhD candidate Zhou Enze** at Huazhong University of Science and Technology (for providing the adaptive training algorithm),
- **Professor Zheng** from the School of Optics and Electronic Information (for computing resources),
- The **[Eclipse Deeplearning4J](https://deeplearning4j.konduit.ai/)** (DL4J) ecosystem (AI library),
- You (for your interest in this project).
