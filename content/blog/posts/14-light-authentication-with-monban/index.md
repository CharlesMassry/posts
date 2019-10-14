---
id: 14
date: "2014-07-08"
title: "Light Authentication With Monban"
---
If you want to have user authentication on Rails, but don't require something as hard hitting as Devise or Clearance, Monban is the way to go. [Monban](https://www.github.com/halogenandtoast/Monban), like Clearance, is also developed by the folks over at [thoughtbot](http://thoughtbot.com). This gem provides some nifty, intuitive DSL.

    authenticate_session(session_params)`  
    sign_in(user)
    signed_in?
    sign_out
    sign_up
    current_user

These methods all do what you would think that they do, except maybe `authenticate_session`, which fetches the necessary keys from strong params during assignment to the user, `user = authenticate_session(session_params)`. You can then call `sign_in` on the result of this, to sign in the user. Monban also provides some neat callback methods, such as `require_login`, which is useful if you want to force user authentication before a particular action.

Unlike Devise, it doesn't provide you with specific Monban views or create large migration file for you. However, it still uses Warden and BCrypt for security, so while it is lightweight, it is by no means underpowered.