---
layout: post
title: "Why it stops"
---

### [Why computers stop] (http://mononcqc.tumblr.com/post/35165909365/why-do-computers-stop)
* The key to providing high availability is to modularize the system so that modules are the unit of failure and replacement.
* The top priority for improving system availability is to reduce administrative mistakes by making self-configured systems with minimal maintenance and minimal operator interaction.
* Bohr vs Heisenbugs

#### what parts of a "system" might lead to failure?
* power
* persistent storage devices
* non-persistent CPU memory
* network
* software: OS, app
* humans

### [Why Internet services fail] (http://www.powershow.com/view/121944-YzRiO/Why_do_Internet_services_fail_and_what_can_be_done_about_it_powerpoint_ppt_presentation)
* operator error is the largest cause of failures
* operator error is the largest contributor to the time to repair
* configuration erros are the largest category of operator errors
* failures in custom-written front-end software are significant
* more extensive online testing and more thoroughly exposing and detecting component failures would benefit a lot
* improvement in the maintenance tools and systems used by service operations staff would decrease time to diagnose and repair problems


### In the two studies, what was the most interesting thing you learned?
Failure and error is as usual as process running.

* We could never avoid failures and errors. Since they are always existing, a right attitude is to make these failures less painful and shorter. And a good design is to assume your software would always fail.
* Human is much more fragile than hardware, and our human operations are full of undeterministic. To compensate this point, a similar approach is to "automate everything". We pre-define the failures and their corresponding auto-recovery strategies, which would be much efficient than relying on human judgments
    * This kind of logic is similar with "do Unit test more than documentation". Documentation needs delicate care to maintain its consistency and clearness, people often get confused with that. On the other hand, reading other's unit test could help with the understanding of how this piece of function works
* I remember the design principle of Erlang is the same: "fails fast".
    * And so is the Go: once one of your goroutine dies, it would grab another goroutine to restart your task. And by default, we could not get the ID of a goroutine, because the design intention of Go is to let us forget about the uniqueness of a goroutine, which is called functional programming that ensure you restart the function confidently
* That is why Site Reliability Engineer could earn as much as software developer

