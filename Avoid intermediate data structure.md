Avoid map data to an intermediate/template data structure when writing, and then map it back when reading.

If there are unavoidable gaps between reading and writing, keep the data structure consistent throughout its lifespan. And ONLY convert it to another data structure when it has to cross the system boundary (e.g., a function, a RESTful API, etc.).

This avoids two costs:

1. We don't need to maintain or understand the redundant data structure until we pass it across the system boundary (maybe an API).
2. It's hard to track the data structure in JavaScript without a type system.