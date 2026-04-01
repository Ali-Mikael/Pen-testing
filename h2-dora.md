# X) Read/Watch/Listen & Summarize
**Objective**
- A few bullet points per section is enough to summarize
- Add your own observation, question or idea

## X.1) [Dora and TLPT testing](<https://terokarvinen.com/buuri-2026-dora-and-threat-lead-penetration-testing/buuri-2026-dora-and-threat-lead-penetration-testing--teros-pentest-course.pdf>)
**DORA**
- Digital Operations Resilience Act
- EU-wide regulation for financial sector resilience
- Requirements for two types of testing:
  1. Basic digital operational resilience testing: vulnerability assesments, source code reviews etc..
  2. Threat-led penetration testing (TLPT)


**TIBER**
- Threat Intelligence-Based Ethical Red Teaming
- TIBER-EU > TIBER-FI
- Following TIBER, a financial entity ensures their project is well-controlled, produces good results and fulfils requirements
- TIBER projects
  - Key roles:
    - Control Team (CT), Control Team Lead (CTL)
    - Threat Intelligence Provider (TIP)
    - Red Team Testers (RTT)
    - Blue Team (BT)


**Common Red Team Blunders**
- Treating the project as test of skill and not moving forward soon enough
- Overconfidence and not tailoring tools enough
- Underestimating the maturity of control mechanisms
- Lack of clear communicationg with the control team

 
**Insight**
- Marko made a good point about being an expert in IT technology before being an expert in testing
- Without having a foundational knowledge of how the systems work, how does one expect to break them?



## X.2 ) [DORA regulation](<https://eur-lex.europa.eu/eli/reg/2022/2554/oj/eng>)
**Article 26: Advanced testing of ICT tools, systems and processes based on TLPT**
- Financial entities have to carry out advanced testing by means of TLTP at least every 3 years (can be increased or reduced)
- Testing have to cover several or all critical functions of a financial entity, and testing shall be performed on live production systems


**Article 27: Requirements for testers for the carrying out of TLPT**
- Testers shall:
  - Be of the highest suitability and reputability
  - Posses expertise in threat intelligence, penetration testing and red team testing
  - Be certified by an accreditation body or adhere to formal codes of conduct or ethical testing
- When using internal testers:
  - Use has been approved by the relevant competent authority or by the single public authority designated in accordance with Article 29(9) and (10)
  - Verification that the financial entity has sufficient dedicated resources and ensured that conflicts of interest are avoided throughout the design and execution phases of the test
  - The threat intelligence provider is external to the financial entity

**Thoughts**
- Don't know how far the **mandated** pen testing has evolved, but curious to see when it's going to land at all companies handling any sensitive customer data

## X.3) [TIBER-FI Procedures and Guidelines](<https://www.suomenpankki.fi/globalassets/bof/en/money-and-payments/the-bank-of-finland-as-catalyst-payments-council/tiber-fi/tiber-fi-2.0-procedures-and-guidelines.pdf>)

**5.4: Testing Phase**
> Note: Excluding all 5.4.x sections as per instructions
- The red-teaming phase consists of two separate process steps, namely: the Red Team Test Plan (RTTP) creation and active testing
- When the RTTP reaches its final stage, stakeholders hold a RTTP meating, where the plan is approved
- Different methologies might then be used to conduct the testing, the document here presents the intrustion kill-chain covered in the previous assigment
- To facilitate more effective testing, and keeping in mind the advantage actual threat actors have as well as time/resource constraints, the entity being tested may deliver additional information to testers to speed up the process etc..
- Should a "leg-up" be provided, it has to be duly logged!


**Thoughts**
- The document illuminates the contrast between ethical hackers and real world threat actors, and how it might effect on test results
- Expanding on this topic, what then becomes the line an ethical hacker should never cross in terms of tools/methologies used?





------




# A) Install Metasploitable
