> developers often use **common parameter names or usernames** to represent an administrator or a user account in URLs, APIs, or databases. Attackers and pentesters test these names when looking for **privilege escalation or IDOR vulnerabilities**.

Below are **50 common names developers use for admin or user identifiers**.

---

# Common Names Used to Indicate Admin or User

### Very Common Admin Names

1. admin
2. administrator
3. root
4. superuser
5. sysadmin
6. superadmin
7. admin_user
8. admin_account
9. main_admin
10. master

---

### Developer / System Style Names

11. system
12. sys
13. operator
14. owner
15. manager
16. moderator
17. support
18. service
19. backend
20. control

---

### Account / Role Based Names

21. account
22. user
23. username
24. userid
25. user_id
26. account_id
27. profile
28. member
29. client
30. customer

---

### Admin Panel Related

31. admin_id
32. admin_user
33. admin_name
34. admin_account
35. administrator_id
36. admin_uid
37. admin_login
38. admin_profile
39. admin_access
40. admin_role

---

### API / Backend Parameter Names

41. uid
42. id
43. user_uuid
44. guid
45. user_guid
46. account_uuid
47. profile_id
48. member_id
49. client_id
50. customer_id

---

# Example Where These Appear

URL parameters:

```
/myaccount?user_id=123
/profile?id=456
/account?uid=789
```

API requests:

```
/api/user?id=123
/api/account?username=admin
```

JSON responses:

```json
{
 "username": "admin",
 "role": "administrator"
}
```

---

# Important Pentesting Idea

Ethical hackers often test:

```
/admin
/admin/login
/admin-panel
/admin-dashboard
```

And parameters like:

```
?id=
?user=
?uid=
?account=
```

Because these commonly expose **IDOR or privilege escalation vulnerabilities**.

---

✅ **Key idea**

Developers frequently use **predictable parameter names**, which makes it easier for attackers to test and manipulate them.

---

Boss, if you want, I can also give you something **extremely useful for web pentesting**:

