---
layout: simple-slide
---

# Код с поочередной подсветкой строк

```js {*|1-3|5-7|9-13|15-21}
function fetchUser(id) {
  return { id, name: "Alice", roles: ["admin", "editor"] };
}

function hasRole(user, role) {
  return user.roles.includes(role);
}

function printUser(user) {
  console.log(`ID: ${user.id}`);
  console.log(`Name: ${user.name}`);
  console.log(`Roles: ${user.roles.join(", ")}`);
}

const user = fetchUser(42);

if (hasRole(user, "admin")) {
  console.log("Access granted");
} else {
  console.log("Access denied");
}
```
