---
title: "Reflecting on the Development of Humadroid: Invitations"
date: 2024-02-01T13:27:00+02:00
draft: false
categories:
  - humadroid
  - Product development
  - Brainstorming
---

Navigating the journey of a solo technical founder comes with its unique set of challenges. From coding to deploying applications, the technical aspects often come naturally. However, the multifaceted roles involving accounting, marketing, design, and notably, product development, present a broader spectrum of challenges. While resources abound for tackling marketing and accounting, discussions around product development are notably scarcer. For small teams or products, the key often lies in the articulation of problems through brainstorming and discussion, a task that typically requires a collaborative effort. Unfortunately, without a product-oriented co-founder, finding alternative avenues to explore and refine ideas becomes crucial. It is in this context that I see blogging as a valuable tool for deliberation.

Today, my focus shifts to the intricacies of invitations and user management within Humadroid—a reflection spurred by a simple request from a friend to test the tool. This request illuminated a critical user experience moment: the confusion a new account holder might face regarding their next steps. Given Humadroid's aim to streamline people management, ensuring user presence within the system is essential. Here, I delve into various approaches for managing this:

1. **Importing Users from Email Systems**
   - **Pros**: This method, implemented via the Google Admin Directory, is reliable for importing users and can be run periodically. Successful imports also confirm domain ownership.
   - **Cons**: It's limited to users within supported systems and assumes the account owner has domain admin rights.
   - **Status**: Implemented.

2. **Inviting Users Individually**
   - **Insight**: While straightforward, this method is impractical for adding more than ten users due to its time-consuming nature and potential for errors.
   - **Status**: Implemented.

3. **Using an Invite Link**
   - **Observation**: Although recently implemented and effective for streamlining sign-ups, it doesn't guarantee that users will complete the registration process.
   - **Status**: Implemented.

4. **Creation of Accounts via OAuth2**
   - **Challenge**: Integrating user accounts with matching email domains presents complexities, especially before verifying domain ownership (consider rogue user registering a account with gmail.com email). This process is complicated for less tech-savvy users (as it requires ie. adding DNS entries) and has been postponed until a feasible verification method is developed.
   - **Status**: Postponed.

5. **Integration with HRSM Systems**
   - **Consideration**: Currently, this is not in the plan as the target customers are in the preliminary stages of adopting HRMS solutions to manage their operations.
   - **Status**: Not planned.

Documenting these reflections not only aids in clarifying the development path but also highlights the necessity of a "verified_domain" feature. This realization guides the prioritization of other upcoming features, such as managing user documents. And clearly I can see the value of blogging as a tool for problem-solving and decision-making in the absence of a collaborative team.
