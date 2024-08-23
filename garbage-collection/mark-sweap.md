# Mark sweep

## Mechanism

1. GC recursively traverse all references from root.
2. Mark all references as traversed.
3. Destroy unmarked references.

Circle reference are no longer a problem.
