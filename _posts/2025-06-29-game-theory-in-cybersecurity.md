---
layout: post
title: "Game Theory in Cybersecurity: Modeling the Adversary"
date: 2025-06-29
---

# Applying Game Theory in Cybersecurity Defense

## Introduction

Cybersecurity is a high-stakes chess match between attackers and defenders, each anticipating the other's next move. In such a dynamic environment, defenders cannot afford to rely on static checklists or reactive playbooks.

**This is where game theory comes in.** Originally rooted in economics and military strategy, game theory provides a mathematical framework to model and analyze strategic interactions, especially useful when you’re constantly trying to outsmart an intelligent, adaptive adversary.

In this post, we’ll explore how defenders can apply game-theoretic thinking to design smarter controls, allocate resources efficiently, and make adversaries work harder for less reward.

---

## Key Types of Games in Cybersecurity

Game theory studies strategic interactions between rational players—each trying to maximize their outcome. In cybersecurity, it helps model attacker-defender relationships in a structured way.

### 1. Zero-Sum Games  
*"My gain is your loss."*

In zero-sum games, the attacker’s success directly equals the defender’s loss. It’s a pure adversarial model with no mutual gain possible.

#### Examples:
- **Ransomware Attack**: If attackers encrypt your data and you pay the ransom, they win (gain money) while you lose (both money and integrity). Your complete loss is their total gain.
- **Credential Theft**: If attackers gain access to admin credentials, the defender loses exclusive control.

> **When to use**: Zero-sum models are useful when modeling direct, binary conflicts—like breach vs. prevention, compromise vs. protection.

---

### 2. Non-Zero-Sum Games  
*"We can both gain—or both lose."*

These games reflect real-world complexity better. Attackers and defenders can both make choices that lead to mutual benefit or mutual loss.

#### Examples:
- **Silent Attack Avoidance**: An attacker skips exploiting a known zero-day because strong EDR will likely detect it. Both parties "win"—you remain uncompromised, and the attacker avoids burning a valuable exploit.
- **Bug Bounty Programs**: The defender pays for responsible disclosure. The attacker (now a researcher) gets rewarded legally. Both benefit.
- **Public Patching**: When vendors publish patches and attackers wait until user adoption is widespread before exploiting (or avoid it entirely), both avoid disruption. This allows time for organizations to protect themselves.
- **Supply Chain Security**: A vendor improves their software integrity pipeline. Attackers may avoid targeting it due to increased complexity, reducing risk for everyone down the chain.

> **When to use**: Use non-zero-sum models when the attacker has incentives to avoid conflict or can benefit from non-malicious behavior.

---

### 3. Stackelberg Games (Leader–Follower)  
*"I move first. You respond."*

In these models, the leader commits to a strategy first, and the follower observes and reacts optimally.

#### Examples:
- **Honeypots and Honeytokens**: The defender plants deceptive assets like fake databases or credentials. The attacker responds—if they touch it, you catch them. The defender *moves first* by shaping the attacker's perception.
- **Firewall Rule Placement**: The defender designs a rule-set (e.g., geo-blocking or port-based blocking), knowing that attackers will probe and adapt accordingly.
- **Publicly Declared Deterrents**: Organizations might publicize their incident response capabilities (e.g., “We have 24/7 SOC monitoring”) to dissuade attackers before they act.
- **Threat Intel Sharing**: When a nation-state discloses an attacker’s TTPs, they force the adversary to adapt, reducing the success of future campaigns.

> **When to use**: Stackelberg games are highly useful in deception, deterrence, and proactive defense scenarios where you want to influence attacker choices in advance.

---

### 4. Bayesian Games (Games with Incomplete Information)  
*"I'm not sure who you are or what you know."*

In these games, players don’t have perfect information about the other’s identity, intentions, or capabilities. Probabilistic reasoning becomes critical.

#### Examples:
- **Phishing Defense**: The defender doesn’t know which employees are targets or when, so they must invest in generalized awareness and filtering. The attacker also doesn’t know how trained or cautious the recipient is.
- **Insider Threat Models**: The organization doesn’t know which insider (if any) is malicious. The threat level is modeled probabilistically.
- **APT Behavior**: Advanced Persistent Threats (APT) often operate stealthily. Defenders must infer their presence from indirect signals, like small anomalies or weak Indicators of Compromise (IoCs).

> **When to use**: Use Bayesian games for modeling uncertainty, especially when risk comes from unknown actors or hidden threats.

---

### 5. Repeated Games  
*"Let’s play again—history matters."*

In repeated games, interactions happen over time. Each round affects future behavior. Trust, reputation, and retaliation come into play.

#### Examples:
- **APT Campaigns**: Nation-state attackers often probe the same target multiple times, adjusting tactics after every detection. Defenders can escalate responses over time.
- **MFA Fatigue Attacks**: If attackers repeatedly send push notifications, defenders may harden policies after seeing multiple failed attempts (e.g., implementing number matching or timing rules).
- **User Behavior Monitoring**: A user who repeatedly accesses sensitive resources outside business hours may trigger tighter access rules in the future.

> **When to use**: Repeated games help model long-term adversarial dynamics and defense evolution.

---

## Payoff Tables with Factors for Key Game Types

### 1. Zero-Sum Game: Ransomware Attack

| Scenario                       | Payoff (Attacker, Defender) | Detection Likelihood | Cost to Attacker | Recovery Cost to Defender | Data Loss      |
|-------------------------------|-----------------------------|---------------------|------------------|---------------------------|----------------|
| Defender has backups           | (0, 0)                      | High                | Medium           | Low                       | None           |
| Defender has no backups        | (10, -10)                   | Medium              | Low              | Very High                 | Total          |
| Attacker chooses not to attack | (0, 0)                      | None                | None             | None                      | None           |

---

### 2. Non-Zero-Sum Game: Noisy Exploit vs. Detection

| Scenario                      | Payoff (Attacker, Defender) | Detection Likelihood | Noise Level | Cost to Attacker | Risk of Attribution |
|------------------------------|-----------------------------|---------------------|-------------|------------------|---------------------|
| Noisy exploit, strong detection | (-2, -1)                  | High                | High        | Medium           | High                |
| Noisy exploit, weak detection   | (6, -6)                   | Low                 | High        | Low              | Low                 |
| Stay silent                   | (0, 0)                      | None                | None        | None             | None                |

---

### 3. Stackelberg Game: Honeypot Strategy

| Scenario                       | Payoff (Attacker, Defender) | False Positive Rate | Detection Accuracy | Defender Cost | Attacker Confusion Level |
|-------------------------------|-----------------------------|---------------------|--------------------|---------------|--------------------------|
| Attacker engages with honeypot| (-5, 5)                     | Low                 | High               | Medium        | High                     |
| Attacker avoids honeypot       | (0, 0)                      | None                | None               | Medium        | Medium                   |
| Defender does not deploy honeypot | (8, -8)                 | N/A                 | N/A                | Low           | None                     |

---

### 4. Bayesian Game: Insider Threat

| User Type      | Defender Strategy           | Payoff (Attacker, Defender) | Detection Capability | Friction to User | Audit Coverage | Potential Damage |
|----------------|----------------------------|-----------------------------|----------------------|------------------|----------------|------------------|
| Benign (80%)   | Strict access controls      | (-1, 4)                     | High                 | Medium           | Full           | None             |
| Benign (80%)   | Open access                | (0, 5)                      | Low                  | Low              | Partial        | None             |
| Malicious (20%)| Strict access controls      | (5, -5)                     | High                 | Medium           | Full           | Limited          |
| Malicious (20%)| Open access                | (10, -10)                   | Low                  | Low              | Partial        | High             |

---

### 5. Repeated Game: MFA Fatigue Attack

| Scenario                       | Payoff (Attacker, Defender) | User Confusion Risk | Defender Response Time | Cost to Attacker | Long-Term Deterrence |
|-------------------------------|-----------------------------|--------------------|-----------------------|------------------|----------------------|
| Repeated MFA spam, no changes | (8, -8)                     | High               | Slow                  | Low              | None                 |
| Repeated MFA spam, policy hardened | (-5, 3)               | Low                | Fast                  | High             | High                 |
| Attacker stops after early detection | (0, 0)              | None               | N/A                   | None             | High                 |

---

## How Are These Payoff Numbers Calculated?

The numbers like **4**, **–1**, **10**, or **–10** in these tables are **conceptual and illustrative**, not exact measurements. Here’s how such numbers are typically derived or modeled:

### 1. Qualitative to Quantitative Translation  
Experts assign numbers based on **relative impact**, **risk**, and **cost** assessments rather than strict formulas. For example:  
- A *high* damage event might be –10 (severe loss).  
- A minor inconvenience might be closer to zero or –1.  
- Positive numbers reflect gains (e.g., +4 means moderate success).

### 2. Utility Functions  
Game theory uses **utility functions** that capture:  
- Financial loss or gain  
- Operational impact  
- Detection likelihood  
- Effort or cost

These are often normalized within ranges like –10 to +10.

### 3. Expert Judgment and Risk Matrices  
Security teams map severity levels to numbers (e.g., low=1, medium=5, high=10) combined with probabilities to estimate expected impact.

### 4. Research and Data-Driven Models  
Academic research or historical incident data can provide estimates, often combining:  
- Cost of incidents  
- Probability of occurrence  
- Expected value calculations

---

## How You Could Calculate or Estimate Payoffs Practically

To rigorously calculate payoffs:  

1. **Define measurable metrics** (e.g., financial cost, downtime hours, likelihood).  
2. **Translate them into utility scores** using consistent scales.  
3. **Incorporate probabilities** of events happening.  
4. **Use formulas like:**

`Expected Payoff = Probability of Event × Impact Score − Cost of Action`

5. **Adjust weights** based on organizational priorities (e.g., brand damage vs. operational disruption).

---

##Note: 

- The payoff values in game theory models are flexible and context-dependent.
- They’re intended to capture **relative preferences and trade-offs**.
- When applying these models, customize scores to your organization’s data and risk appetite.

---


