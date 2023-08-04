---
slug: 2023-06-02-release-notes
title: Release Notes May 2023
authors: yana
tags: [release-notes, updates]
---

Beda EMR Release Notes. May 2023.

👇

<!--truncate-->

Clinical Forms Management
- Released Chat-based ChatGPT-driven Clinical Forms builder
We wanted to create a Clinical Forms builder for non-technical users. We aimed for a clean and not-overwhelming user experience.
It’s finally here. Creating Forms is as easy as typing “a questionnaire to assess depression”.
The builder comes with the advanced mode. It gives the flexibility to fine-tune the parameters of a particular question and answer options in a convenient drag-n-drop way.
- Added support for ValueSets and Concepts in the Clinical Forms.
When the user configures the Clinical Form questions with a choice-type answer, it is possible to define which ValueSets and Concepts will be used as answer options. An example of usage: picking a disease from the ICD11 catalog.

Scheduling module
- Introduced the ability to publish an available schedule for non-authorized users. It helps patients to book appointments without registration on the patient portal. Additionally, it simplifies integration with external booking platforms.
- Improved scheduling capabilities for practitioners and medical facilities (custom slots, flexible availability for different days etc)
- Implemented re-scheduling workflow

Security and Access control
- Implemented Audit log for the medical documents history

Practitioner UI
- Added a new compact Patient Summary view for quick access to important data.