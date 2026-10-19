## Q&A

### Why `$$` in docker-compose?

Docker Compose replaces `${VAR}` **before** the container starts.

But you want the variable to be expanded **inside the container** (from `env_file`).

So:

* `${VAR}` → Compose substitutes it
* `$${VAR}` → Compose escapes `$` and passes `${VAR}` to the container
* Then the container’s shell expands it

**In short:**
`$$` = “Don’t substitute now. Let the container handle it.”

---

test: ["CMD-SHELL", "pg_isready -U $${POSTGRES_USER} -d $${POSTGRES_DB}"]


Here for the string based command there required to use $${POSTGRES_USER} for the resolving
