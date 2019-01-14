In almost all the blogs and books you will find drawings about concentric circles representing different areas of software. As Robert C. Martin explains in his “Clean Architecture” post, the outer circle is where your infrastructure resides. The inner circle is where your Entities live. The overriding rule that makes this architecture work is The Dependency Rule. This rule says that source code dependencies can only point inwards. Nothing in an inner circle can know anything at all about something in an outer circle.



