---
layout: post
status: publish
published: true
title: GOTO Not Always Considered Harmful
author: noah
author_login: noah
author_email: noah@hackerhasid.com
date: 2012-07-02 19:51:55.000000000 -04:00
categories:
- Uncategorized
tags: []
comments: []
---
Many if not most developers have a certain ingrained contempt for the goto statement available in many languages. I'll contend though that there are times it can aid readability and its existence in modern programming languages is useful.

Take the following code snippet:
{% highlight php %}

function gc_auth_login_validate($form, &$form_state) {
  if ($account) {
    // drupal user exists
    $roles = array_values($account->roles);
    if (in_array(GC_AUTH_ROLE_NAME, $roles)){
      // this is a gc user, authenticate against gc
      $max_login_attempts = $account->data[GC_AUTH_MAX_LOGIN_ATTEMPTS_VARIABLE_NAME] ?: variable_get(GC_AUTH_MAX_LOGIN_ATTEMPTS_VARIABLE_NAME);
      if (gc_auth_check_user($account->mail, $password, $max_login_attempts)->success) {
        goto VALID_GC_CREDENTIALS;
      } else {
        gc_auth_increment_user_login_attempts($account);        
      }
    }    
  } else {
    // no drupal user exists
    // check for a Group Commerce API user
    // ['values']['name'] contains a username-ified version of the email address
    // ['input']['name'] contains the actual email address inputted
    $email = $form_state['input']['name'];

    $response = gc_auth_check_user($email, $password);
    if ($response->success) {
      $account = gc_auth_create_corresponding_drupal_user($email, $response->userKey);
      if (!$account) {
        // something went wrong creating the corresponding account
        form_set_error(NULL, t('An error occurred.  Please try again or contact support.'));
        return;
      }
      goto VALID_GC_CREDENTIALS;
    }
  }
  VALID_GC_CREDENTIALS:
  // reset max login attempts on user
  user_save($account, array(
    'data' => array(
      GC_AUTH_MAX_LOGIN_ATTEMPTS_VARIABLE_NAME    => variable_get(GC_AUTH_MAX_LOGIN_ATTEMPTS_VARIABLE_NAME),
      GC_AUTH_LOGINS_ATTEMPTED_KEY                => 0,
      )
    )
  );
  $form_state['uid'] = $account->uid;
  user_login_submit(array(), $form_state);
  return;

  NOT_A_GC_USER:
  // form is empty or this is not a gc user, authenticate using the default validators
  foreach($form_state['values']['validators'] as $validator) {
    $validator($form, $form_state);    
  }
  return;
}

{% endhighlight %}

In this case, the goto statement prevents having to refactor into its own method and enables me to easily code multiple branches that end up performing the same functionality.  The alternative would have been to refactor into its own method which is unwanted in this case since the functionality belongs with the login functionality.
