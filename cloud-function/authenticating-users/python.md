---
title: Authenticating Users
---

<a name="getting-current-user-id"></a>
## Getting the current user ID

In your cloud code, you can obtain the user ID of the authenticated user by
calling the function `current_user_id` in the `skygear.utils.context` module.

The current user ID will be useful in database hooks, lambda functions, or
custom HTTP endpoint (if authenticated). Scheduled tasks are not run with
any authenticated user so you cannot get an authenticated user from there.

```python
from skygear.utils.context import current_user_id

@skygear.op('create-task', user_required=True)
def create_task(description):
    user_id = current_user_id()
    # do further processing with user_id
```

<a name="reset-password"></a>
## Reset Password

You can reset (change) the password of a user using the 
`reset_password_by_username` function in the `skygear.utils.user` module.

```python
import skygear
from skygear.utils.user import reset_password_by_username

@skygear.op('my-reset-password')
def custom_reset_password(username, new_password):
    # also need to check permission so that only admin can use this
    is_success = reset_password_by_username(username, new_password)
    return {
        'success': is_success,
    }
```

### Parameters

- **username** (String)

  The username whose password is to be changed.

- **new_password** (String)

  The new password to be assigned to the user.

### Return Value

The function returns `True`
if the password is successfully changed, otherwise it returns `False`.
