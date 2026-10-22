# Photogram Industrial, Part 4: Building Out the Views

## Getting started

Let's continue building our Photogram Industrial project. Here's the target we're working towards:

[pg-industrial.matchthetarget.com](https://pg-industrial.matchthetarget.com/)

Navigate to `github.com/codespaces` (or reopen the previous lesson and use the "Load assignment" button) and reopen your `pg-industrial` project codespace to continue building on what you accomplished in the previous lessons.

At this point, you should have a navigable app with working authentication, a responsive three-column layout with sidebar navigation, phone navigation, a photo upload modal, customized scaffold controllers, and a `UsersController` with stub view templates. What we _don't_ have yet are the real view templates: the photo card, the feed, the user profile page, the follower lists, and customized Devise views. That's what we'll build in this lesson.

Make sure your sample data is loaded before continuing:

```
rake sample_data
```

Since each lesson builds on the work from the previous one, we want to branch off of our last branch rather than `main`. First, switch to the branch from the previous lesson:

```
git switch routes-layout-controllers
```

Now create a new branch from there:

```
git switch -c profile-page-and-views
```

## Follow/unfollow partial

Almost every [page in the target](https://pg-industrial.matchthetarget.com/) shows a Follow, Requested, or Following button next to usernames. Let's build that reusable partial first, so it's ready when we need it.

The partial handles three states for the relationship between two users:

1. **Following**: the sender already follows the recipient (an accepted follow request).
2. **Requested**: the sender has sent a follow request that is still pending.
3. **Follow**: no follow request exists yet.

Create `app/views/follow_requests/_follow_unfollow.html.erb`:

```erb
<% unless sender == recipient %>
  <% follow_request = sender.sent_follow_requests.find_by(recipient: recipient) %>

  <% if follow_request %>
    <% if follow_request.pending? %>
      <%= button_to follow_request, method: :delete, class: "btn btn-primary rounded-full icon-link" do %>
        <i class="fa-solid fa-envelope"></i>
        Requested
      <% end %>
    <% elsif follow_request.accepted? %>
      <%= button_to follow_request, method: :delete, class: "btn btn-primary rounded-full icon-link" do %>
        <i class="fa-solid fa-check"></i>
        Following
      <% end %>
    <% end %>
  <% else %>
    <%= render "follow_requests/form", follow_request: recipient.received_follow_requests.build %>
  <% end %>
<% end %>
```
{: filename="app/views/follow_requests/_follow_unfollow.html.erb" copyable }

Let's walk through it:

- `unless sender == recipient`: you shouldn't see a follow button next to your own name. `unless` is Ruby's opposite of `if`: the code inside runs only when the condition is `false`. It means the same as `if sender != recipient`, or `if !(sender == recipient)`.
- We look up whether the sender already has a follow request for this recipient with `find_by`, which returns the record or `nil`.
- If a request exists and is `pending?`, we show a "Requested" button with an envelope icon. If it's `accepted?`, we show a "Following" button with a check icon. Both are `button_to`s that send a DELETE request, so clicking either one cancels the request or unfollows. The `pending?` and `accepted?` methods come for free from our `enum :status` declaration on the FollowRequest model.
- If no request exists, we render the follow request form.

The last `render` passes the form a new follow request, made with `recipient.received_follow_requests.build`. Just like `current_user.own_photos.build` in [the photo modal](/lessons/839-pg-industrial-3#new-photo-modal), `.build` makes a new, unsaved record through an association, so the new follow request already has its `recipient_id` filled in.

Here's a check on `.build`.

- In a library app, what does `author.books.build` return?
- All of the author's books
  - Not quite. That's `author.books` on its own.
- A new book, saved to the database
  - Not quite. `.build` doesn't save; that's `.create`.
- A new, unsaved book, with its `author_id` already set
  - Yes! Calling `.build` through an association fills in the foreign key.
{: .choose_best #build_through_association title="Building through an association" points="1" answer="3" }

<div class="alert alert-info">

<i class="fa-solid fa-circle-info"></i> We'll be creating lots of view templates like this one, full of conditional logic and helper methods. You can copy them, but read every line before you move on, and consider typing out at least some of them to help with your understanding. If you don't understand something, stop and ask questions!
</div>

### Follow request form

Replace the scaffold-generated `app/views/follow_requests/_form.html.erb`:

```erb
<%= form_with(model: follow_request) do |form| %>
  <%= form.hidden_field :recipient_id %>

  <%= form.button class: "btn btn-primary rounded-full icon-link" do %>
    <i class="fa-solid fa-plus"></i>
    Follow
  <% end %>
<% end %>
```
{: filename="app/views/follow_requests/_form.html.erb" copyable }

The form only sends the `recipient_id`, which `.build` filled in. We use `form.button ... do ... end` (the block form) to put both a Font Awesome icon and the text inside the button. `btn btn-primary` makes it a solid button, `rounded-full` rounds its ends, and `icon-link` lines the icon up with the text.

Recall from the previous lesson that `FollowRequestsController#create` accepts the request right away if the recipient's account is public. So clicking "Follow" on a public account changes the button to "Following" when the page reloads.

Commit:

```
git add -A
git commit -m "Added follow/unfollow partial and form"
git push --set-upstream origin profile-page-and-views
```

That last command publishes the branch to GitHub. From now on, you can push with just `git push` after each commit.

### Open and submit your pull request

Now that you've pushed your branch to GitHub, it's time to open a **pull request** (PR). A pull request lets us review your code and leave line-by-line feedback. ([Step 3 of The Pull Request Workflow](/lessons/936-pull-request-workflow#step-3-open-a-pull-request) has the general steps.)

<div class="alert alert-primary">

<i class="fa-solid fa-circle-play"></i> [Here is a short video demonstration of the process.](https://share.descript.com/view/RLP4apAu5pp) You should also carefully read the notes below!
</div>

On GitHub, navigate to your `pg-industrial` repository. You should see a prompt to open a pull request for the `profile-page-and-views` branch that you just published, or you can go to the "Pull requests" tab and click "New pull request."

Make sure the **base** branch is `routes-layout-controllers` and the **compare** branch is `profile-page-and-views`. Also be sure to change the base _repository_ from `appdev-projects/pg-industrial` to _your_ fork. Give it a title, then click "Create pull request."

<aside>

If there's no base _repository_ dropdown when you open your pull request, skip that step. Projects launched from the Grades page now save your work to your own repository rather than a fork, so your pull request already targets your repository.
</aside>

Your PR URL should look like:

```
github.com/[YOUR_GITHUB_USERNAME]/pg-industrial/pull/X
```

It should _not_ contain `appdev-projects` in the URL. If it does, you submitted a PR to _our_ repo instead of to _your own fork_.

<aside>
You don't need to merge your branches now, but when you're ready: [see Step 6 of The Pull Request Workflow](/lessons/936-pull-request-workflow#step-6-squash-and-merge-to-main).
</aside>

Any additional commits you push to this branch will automatically show up on the pull request. As you work through the rest of this lesson, you can periodically check the PR diff on GitHub in the "Files changed" tab to get a comprehensive overview of everything that has changed so far.

Submit your pull request URL:

- `profile-page-and-views` compared to `routes-layout-controllers`:
- github.com
  - Great job!
- any
  - Not quite. Make sure the URL looks like: `github.com/[YOUR_GITHUB_USERNAME]/pg-industrial/pull/X`
{: .free_text #pr_url title="Pull request URL" points="1" answer="1" }

## Comment partial

Comments appear below every photo. Let's build the comment itself, and then the form for adding a new one.

Each comment shows the author's avatar, display name, username, how long ago it was posted, the comment body, and a `...` menu with Edit and Delete. Replace the scaffold-generated `app/views/comments/_comment.html.erb`:

```erb
<li>
  <div class="flex gap-3">
    <%= image_tag comment.author.avatar_image, class: "rounded-full img-small img-cover shrink-0" %>

    <div class="grow">
      <div>
        <%= link_to user_path(comment.author.username) do %>
          <%= comment.author.display_name %>
          <span class="font-light text-zinc-500">@<%= comment.author.username %></span>
        <% end %>

        <span class="font-light text-zinc-500">
          &middot;
          <%= time_ago_in_words(comment.created_at) %>
        </span>
      </div>

      <p class="mb-0">
        <%= comment.body %>
      </p>
    </div>

    <div class="dropdown" data-controller="dropdown">
      <button type="button" class="btn btn-link" data-action="click->dropdown#toggle" aria-expanded="false">
        <i class="fa-solid fa-ellipsis"></i>
      </button>

      <ul class="dropdown-menu right-0" data-dropdown-target="menu">
        <li>
          <%= link_to "Edit", edit_comment_path(comment), class: "dropdown-item" %>
        </li>
        <li>
          <%= button_to "Delete", comment_path(comment), method: :delete, class: "dropdown-item" %>
        </li>
      </ul>
    </div>
  </div>
</li>
```
{: filename="app/views/comments/_comment.html.erb" copyable }

Can you match up the opening and closing tags? The comment is an `<li>`, because we'll list a photo's comments inside a `<ul>`. Inside it, a flex row (`flex gap-3`) holds three things side by side:

1. The avatar. `img-small` and `img-cover` come from `app/assets/stylesheets/custom-image.css`, which is already in your project: `img-small` makes the image 36 pixels square, and `img-cover` crops it to fill that square without stretching. `shrink-0` stops the row from squeezing it.
2. The author and the comment body, in a `<div>` with `grow`, so it takes up all the room the other two leave.
3. The `...` menu. This is the same `dropdown` Stimulus controller that opens the user menu in the left sidebar. `right-0` lines the menu up with the right edge of the button, so that it doesn't run off the side of the page.

`time_ago_in_words` turns a time into words like "5 minutes" or "about 2 hours."

<aside>

`&middot;` is an HTML _entity_: a code that the browser displays as a character. Entities start with `&` and end with `;`, and `&middot;` displays a middle dot (·). They're handy for characters that are hard to type, and for the characters that HTML itself uses, like `&lt;` for `<`. MDN has [a list of them](https://developer.mozilla.org/en-US/docs/Glossary/Character_reference).
</aside>

## Comment form

Replace the scaffold-generated `app/views/comments/_form.html.erb`:

```erb
<%= form_with(model: comment) do |form| %>
  <%= form.hidden_field :photo_id %>

  <div class="form-group">
    <%= form.label :body, class: "visually-hidden" %>
    <%= form.text_area :body, class: "form-control", placeholder: "Add a comment" %>
  </div>

  <%= form.submit class: "btn btn-primary" %>
<% end %>
```
{: filename="app/views/comments/_form.html.erb" copyable }

The `photo_id` goes in a hidden field, since the photo is filled in when we build the comment, and the label is `visually-hidden`: screen readers still read it, but the placeholder takes its place on the screen. That leaves a text area and a submit button.

## Like/unlike partial

The like button switches between a solid heart (you've liked the photo) and an outline heart (you haven't). Let's build both states.

Create `app/views/photos/_likes.html.erb`:

```erb
<% like = current_user.likes.find_by(photo: photo) %>

<% if like %>
  <%= button_to like, method: :delete, class: "btn btn-link icon-link" do %>
    <i class="fa-solid fa-heart"></i>
    <%= pluralize(photo.likes_count, "like") %>
  <% end %>
<% else %>
  <%= render "likes/form", like: photo.likes.build(fan: current_user) %>
<% end %>
```
{: filename="app/views/photos/_likes.html.erb" copyable }

The logic is:

1. Look up whether the current user has already liked this photo.
2. If they have, show a **solid heart** (`fa-solid fa-heart`) in a `button_to` that sends a DELETE request to destroy the like.
3. If they haven't, render the like form, with an **outline heart** that creates a new like.

`photo.likes.build(fan: current_user)` is `.build` again, this time with an argument: it fills in the `photo_id` because we call it through `photo.likes`, and the `fan` because we pass it. `pluralize` puts the right word after the count: "1 like" or "3 likes."

### Like form partial

Replace the scaffold-generated `app/views/likes/_form.html.erb`:

```erb
<%= form_with(model: like) do |form| %>
  <%= form.hidden_field :photo_id %>

  <%= form.button class: "btn btn-link icon-link" do %>
    <i class="fa-regular fa-heart"></i>
    <%= pluralize(like.photo.likes_count, "like") %>
  <% end %>
<% end %>
```
{: filename="app/views/likes/_form.html.erb" copyable }

The `photo_id` is sent in a hidden field so that `LikesController#create` knows which photo to like. The fan isn't sent at all: the controller sets it from `current_user`, as we saw in the previous lesson. `btn-link` makes the button look like a link rather than a solid button.

### Like partial

Replace the scaffold-generated `app/views/likes/_like.html.erb`:

```erb
<li>
  <div class="flex gap-3">
    <%= image_tag like.fan.avatar_image, class: "rounded-full img-small img-cover shrink-0" %>

    <div class="grow">
      <div class="flex items-center justify-between">
        <%= link_to user_path(like.fan.username) do %>
          <%= like.fan.display_name %>
          <span class="block font-light text-zinc-500">@<%= like.fan.username %></span>
        <% end %>

        <%= render "follow_requests/follow_unfollow", sender: current_user, recipient: like.fan %>
      </div>

      <p class="mb-0">
        <%= like.fan.bio %>
      </p>
    </div>
  </div>
</li>
```
{: filename="app/views/likes/_like.html.erb" copyable }

We'll use this partial on [the "Liked by" page](https://pg-industrial.matchthetarget.com/photos/10/likes), which lists everyone who liked a photo. It has the same shape as a comment: the avatar, then the fan's name and username, with a follow button pushed to the right by `justify-between`.

Now let's commit all the partials we've created so far:

```
git add -A
git commit -m "Added comment, like, and supporting partials"
git push
```

## The photo card partial

The photo card is the most important partial in the app. Every photo on the feed, discover, and profile pages uses it. Here's a finished one, on Alice's profile page:

![A photo card on Alice's profile page, with numbered callouts on its parts](/assets/photo-card-annotated.png)

1. The "Pinned" badge (only on the owner's profile page).
2. The owner's avatar and username, and a follow button.
3. The photo.
4. The like button and the comment count.
5. The `...` menu: Edit, Delete, and Pin.
6. The number of likes, which links to the "Liked by" page.
7. The owner's name, how long ago they posted, and the caption.
8. The comments.
9. The form for a new comment.

Replace the scaffold-generated `app/views/photos/_photo.html.erb`. We'll build this file section by section, so delete what's in it now.

### Pinned badge

First, the card itself and the "Pinned" badge:

```erb
<div class="card">
  <% if photo.pinned? && current_page?(user_path(photo.owner.username)) %>
    <div class="icon-link text-blue-600 mb-2">
      <i class="fa-solid fa-thumbtack"></i>
      Pinned
    </div>
  <% end %>
<!-- ... -->
```
{: filename="app/views/photos/_photo.html.erb" }

`class="card"` turns the `<div>` into a white box with a border and rounded corners, like the cards in [the AppDev styles reference](/lessons/945-style-with-appdev-tailwind#cards).

We only show the "Pinned" badge on the owner's profile page. The `current_page?` helper checks whether the URL of the current request matches the path we give it. Pinned photos also go to the top of the owner's profile page, as we'll see.

You can try pinning in [the target](https://pg-industrial.matchthetarget.com/): sign in, visit your profile page, and use the `...` menu on one of your photos to "Pin" or "Un-pin" it. The thumbtack and "Pinned" don't show on the feed or other pages, where `current_page?(user_path(photo.owner.username))` is `false`.

### Owner and follow button

Next, the owner's avatar and username, and a follow button:

```erb{4-8,10-11}
<!-- ... -->
  <% end %>

  <div class="flex items-center justify-between mb-2">
    <div class="flex items-center gap-2">
      <%= image_tag photo.owner.avatar_image, class: "rounded-full img-small img-cover" %>
      <%= link_to photo.owner.username, user_path(photo.owner.username), class: "font-semibold" %>
    </div>

    <%= render "follow_requests/follow_unfollow", sender: current_user, recipient: photo.owner %>
  </div>
<!-- ... -->
```
{: filename="app/views/photos/_photo.html.erb" }

We reuse the `follow_requests/follow_unfollow` partial, with `current_user` as the `sender` and the photo's owner as the `recipient`.

### Image and action buttons

Now the photo, and below it a row with the like button, a comment count, and the `...` menu:

```erb{4,6-8,10-14,16-19,21-33}
<!-- ... -->
  </div>

  <%= image_tag photo.image, class: "w-full rounded-lg" %>

  <div class="flex items-center justify-between">
    <div class="flex">
      <%= render "photos/likes", photo: photo %>

      <%= link_to photo_path(photo), class: "btn btn-link icon-link" do %>
        <i class="fa-regular fa-comment"></i>
        <%= photo.comments_count %>
      <% end %>
    </div>

    <div class="dropdown" data-controller="dropdown">
      <button type="button" class="btn btn-link" data-action="click->dropdown#toggle" aria-expanded="false">
        <i class="fa-solid fa-ellipsis"></i>
      </button>

      <ul class="dropdown-menu right-0" data-dropdown-target="menu">
        <li>
          <%= link_to "Edit", edit_photo_path(photo), class: "dropdown-item" %>
        </li>
        <li>
          <%= button_to "Delete", photo_path(photo), method: :delete, class: "dropdown-item" %>
        </li>
        <li>
          <%= button_to photo.pinned? ? "Un-pin" : "Pin", photo_path(photo), method: :patch, class: "dropdown-item", params: { photo: { pinned: !photo.pinned } } %>
        </li>
      </ul>
    </div>
  </div>
<!-- ... -->
```
{: filename="app/views/photos/_photo.html.erb" }

`w-full` stretches the photo to the width of the card, and `rounded-lg` rounds its corners. On the left of the row, we render the like button partial and a comment icon that links to the photo's page. The `...` menu on the right is the same dropdown as the one on each comment, with three options:

- **Edit** links to the photo's edit page.
- **Delete** sends a DELETE request with `button_to`.
- **Pin** or **Un-pin** sends a PATCH request that flips the photo's `pinned` column.

#### Pinning with `button_to` and `params:`

The pin line is the trickiest one, so let's unpack it:

```erb
<%= button_to photo.pinned? ? "Un-pin" : "Pin", photo_path(photo), method: :patch, class: "dropdown-item", params: { photo: { pinned: !photo.pinned } } %>
```

We already know that `button_to` makes a small form with one button, which submits to a URL. Here:

- `photo.pinned? ? "Un-pin" : "Pin"` is the button's text. This is Ruby's **ternary operator**, a one-line shortcut for an `if`/`else`: `condition ? value_if_true : value_if_false`. If the photo is pinned, the button says "Un-pin"; otherwise, it says "Pin."
- `photo_path(photo)` is the photo's URL (e.g., `/photos/42`), and `method: :patch` sends a PATCH request there, which reaches the `update` action in `PhotosController`.
- `params: { photo: { pinned: !photo.pinned } }` adds hidden fields to the form, without our having to write them out. The nested hash means the parameter arrives as `photo[pinned]`, which is the shape that `photo_params` expects in the controller (`params.expect(photo: [ :image, :caption, :pinned ])`).
- `!photo.pinned` flips the boolean. If the photo is pinned (`true`), we send `false` to un-pin it, and if it isn't, we send `true`. One button, two behaviors.

Here's a check on `button_to` with `params:`.

- A post card has `<%= button_to "Archive", post_path(post), method: :patch, params: { post: { archived: true } } %>`. What does clicking it send?
- A GET request to `/posts/42?archived=true`
  - Not quite. `method: :patch` makes it a PATCH request, and `params:` become hidden fields in the form.
- A PATCH request to `/posts/42`, with `post[archived]` set to `true`
  - Yes! The nested hash arrives in the shape that `post_params` expects.
- A PATCH request to `/posts/42`, with no params
  - Not quite. `params:` adds hidden fields to the form that `button_to` makes.
{: .choose_best #button_to_params title="Extra params with button_to" points="1" answer="2" }

### Likes count and caption

Next, the number of likes, and the caption with the owner's name and how long ago they posted:

```erb{4-7,9-13,15-19,21-23}
<!-- ... -->
  </div>

  <div class="mb-1">
    <%= link_to photo.likes_count, photo_likes_path(photo) %>
    <%= "like".pluralize(photo.likes_count) %>
  </div>

  <div class="mb-1">
    <%= link_to user_path(photo.owner.username) do %>
      <%= photo.owner.display_name %>
      <span class="font-light text-zinc-500">@<%= photo.owner.username %></span>
    <% end %>

    <span class="font-light text-zinc-500">
      &middot;
      <%= time_ago_in_words(photo.created_at) %>
    </span>
  </div>

  <p>
    <%= photo.caption %>
  </p>
<!-- ... -->
```
{: filename="app/views/photos/_photo.html.erb" }

The number of likes links to the photo's "Liked by" page, through the nested route `/photos/:photo_id/likes` from the previous lesson. This time we only want the count to be a link, so we use the `pluralize` method on the word itself, `"like".pluralize(photo.likes_count)`, which returns just "like" or "likes."

### Comments and the comment form

Finally, add the photo's comments and the comment form to the very bottom of the file:

```erb{4-6,8-9}
<!-- ... -->
  </p>

  <ul class="mb-4">
    <%= render photo.comments.default_order %>
  </ul>

  <%= render "comments/form", comment: photo.comments.build %>
</div>
```
{: filename="app/views/photos/_photo.html.erb" }

The comments are listed in a `<ul>`, which is why each comment partial is an `<li>`. Below them is the comment form, with a new comment built through `photo.comments`.

#### Rendering a collection

Let's zoom in on this line:

```erb
<%= render photo.comments.default_order %>
```

`photo.comments.default_order` uses the `default_order` scope from an earlier lesson to list the comments oldest first. To see what `render` does with them, imagine writing it the long way, with an `each` loop:

```erb
<% photo.comments.default_order.each do |comment| %>
  <%= render partial: "comments/comment", locals: { comment: comment } %>
<% end %>
```

That renders the `comments/_comment.html.erb` partial once for each comment, and gives it a local variable called `comment`. It works, but Rails has a shortcut for it, called **collection rendering**:

```erb
<%= render partial: "comments/comment", collection: photo.comments.default_order %>
```

The `collection:` option tells Rails: _"render this partial once for each item in the collection, and each time, set a local variable named after the partial (`comment`) to the current item."_

And because this pattern is so common, Rails lets us shorten it even further:

```erb
<%= render photo.comments.default_order %>
```

When you pass a collection of records directly to `render`, Rails works out the partial from the model (`Comment` → `comments/_comment`) and names the local variable `comment`. All three versions produce the same HTML.

Here's a check on collection rendering.

- In a bookstore app, which partial does `<%= render @books %>` render for each book?
- `books/index.html.erb`
  - Not quite. Collection rendering renders a partial, and partial names start with an underscore.
- `books/_book.html.erb`
  - Yes! Rails takes the folder and the partial's name from the model, `Book`.
- `shared/_book.html.erb`
  - Not quite. Rails looks in the folder named after the model, `books`.
{: .choose_best #collection_rendering title="Collection rendering" points="1" answer="2" }

<div class="alert alert-success">

<i class="fa-solid fa-circle-check"></i> If your server isn't running, start it with `bin/dev`. Sign in (did you `rake sample_data`?) and visit `/photos/1`. Compare it with [the same page in the target](https://pg-industrial.matchthetarget.com/photos/1). The photo card and its comments should match the target.
</div>

If everything looks good, commit:

```
git add -A
git commit -m "Built photo card partial"
git push
```

## Feed page

Let's put the photo card to use. Replace the stub in `app/views/users/feed.html.erb`:

```erb
<% content_for :title, "Feed" %>

<h1>Feed</h1>

<%= render @photos %>
```
{: filename="app/views/users/feed.html.erb" }

That's collection rendering again: one photo card for each photo in `@photos`. The `@photos` variable comes from the `UsersController#feed` action:

```rb{3}
  # ...
  def feed
    @photos = current_user.feed.latest
  end
  # ...
```
{: filename="app/controllers/users_controller.rb" }

and `current_user.feed` comes from the `has_many :through` association that we defined on the User model:

```rb
  # ...
  has_many :feed, through: :leaders, source: :own_photos
  # ...
```
{: filename="app/models/user.rb" }

So many things come together in that one `render @photos` line: a route, a controller action, an association that joins three tables, the `latest` scope from Part 1 that puts the newest photos first, and a partial that renders four more. Does it all make sense based on what you've learned so far? This is a great time to pause and make sure you can trace every piece of it.

Ask questions now!

<div class="alert alert-success">

<i class="fa-solid fa-circle-check"></i> Visit `/`. You should see photo cards with images, like buttons, comment forms, and follow buttons. Add a comment, and see it appear below the photo. Un-like and like a photo.
</div>

![The feed page, signed in as Alice, with photo cards in the center column](/assets/feed.png)

Commit:

```
git add -A
git commit -m "Built feed page with photo cards"
git push
```

## Discover page

The discover page shows photos liked by the people you follow. Replace the stub in `app/views/users/discover.html.erb`:

```erb
<% content_for :title, "Discover" %>

<h1>Discover</h1>

<%= render @photos %>
```
{: filename="app/views/users/discover.html.erb" }

The template is the same as the feed's, and so is the photo card. The difference is where `@photos` comes from: `feed` goes from a user to their leaders to the leaders' own photos, while `discover` goes from a user to their leaders to the photos the leaders liked. The associations we defined in an earlier lesson write all of that SQL for us.

<div class="alert alert-success">

<i class="fa-solid fa-circle-check"></i> Visit `/discover`. You should see the photos liked by the people you follow, as photo cards.
</div>

Commit:

```
git add -A
git commit -m "Built discover page"
git push
```

## User profile page

Visit `/alice` in [the target](https://pg-industrial.matchthetarget.com/alice). Here's what we'll build:

![Alice's profile page, with numbered callouts on its parts](/assets/profile-annotated.png)

1. The profile banner.
2. The avatar, which overlaps the banner.
3. The display name, a "Private" badge on private accounts, and a follow button.
4. The username.
5. The followers, following, and posts counts. (On your own private account, there's a "pending" count too.)
6. The bio and website.
7. The Posts and Likes tabs, with pinned photos first.

The route `get ":username" => "users#show", as: :user` sends URLs like `/alice` to the `UsersController#show` action, and the `set_user` `before_action` finds the user with `find_by!(username: params.fetch(:username))`. Can you trace that from `config/routes.rb` to `app/controllers/users_controller.rb`?

Once you have, we can replace the stub in `app/views/users/show.html.erb`. We'll build this file section by section, so delete what's in it now.

### Title, banner, and avatar

Start with the page title, the profile banner, and the avatar:

```erb
<% content_for :title, "@#{@user.username}'s profile" %>

<div class="h-48 rounded-lg bg-zinc-200 bg-cover bg-center" style="<%= "background-image: url('#{url_for(@user.profile_banner)}')" if @user.profile_banner.attached? %>"></div>

<%= image_tag @user.avatar_image, class: "rounded-full img-cover img-medium border-4 border-white -mt-16 ml-4" %>
<!-- ... -->
```
{: filename="app/views/users/show.html.erb" }

The banner is an empty `<div>`, 12rem tall (`h-48`), with the banner image as its CSS `background-image`. We only add the `style` attribute when the user has a banner attached, and `url_for` gives us the image's URL. Without a banner, the `<div>` is light gray (`bg-zinc-200`). `bg-cover bg-center` scales the image to fill the `<div>`, and keeps its middle in view.

The avatar has a white border (`border-4 border-white`), and `-mt-16` gives it a _negative_ top margin, which pulls it up over the bottom of the banner. We don't check `attached?` for the avatar, because the `before_create :set_default_avatar` callback gives every user one.

<div class="alert alert-success">

<i class="fa-solid fa-circle-check"></i> Visit `/alice`. You should see the banner and a round avatar overlapping it.
</div>

### Display name, private badge, and follow button

Next, the display name, with a "Private" badge and the follow button on the right:

```erb{4-7,9-15,17-19,21-23}
<!-- ... -->
<%= image_tag @user.avatar_image, class: "rounded-full img-cover img-medium border-4 border-white -mt-16 ml-4" %>

<div class="flex items-center justify-between">
  <h1 class="mb-0">
    <%= @user.display_name || @user.username %>
  </h1>

  <div class="flex items-center gap-2">
    <% if @user.private %>
      <span class="badge text-bg-light">
        Private
        <i class="fa-solid fa-lock"></i>
      </span>
    <% end %>

    <%= render "follow_requests/follow_unfollow", sender: current_user, recipient: @user %>
  </div>
</div>

<p class="text-lg text-zinc-500">
  @<%= @user.username %>
</p>
<!-- ... -->
```
{: filename="app/views/users/show.html.erb" }

We display the `display_name` if the user has one, and fall back to the `username` with `||` if they don't. `badge` is a component class from our stylesheet for a small label, and `text-bg-light` gives it a light background.

### Profile stats

Add the followers, following, pending, and posts counts:

```erb{4-7,9-11,13-17,19-22}
<!-- ... -->
</p>

<div class="flex flex-wrap gap-4 mb-4">
  <%= link_to followers_path(@user.username) do %>
    <strong><%= @user.followers.count %></strong> followers
  <% end %>

  <%= link_to follows_path(@user.username) do %>
    <strong><%= @user.leaders.count %></strong> following
  <% end %>

  <% if current_user == @user && @user.private? %>
    <%= link_to pending_path do %>
      <strong><%= @user.pending_received_follow_requests.count %></strong> pending
    <% end %>
  <% end %>

  <span>
    <strong><%= @user.photos_count %></strong> posts
  </span>
</div>
<!-- ... -->
```
{: filename="app/views/users/show.html.erb" }

A few details to notice:

- `link_to ... do ... end` (the block form) puts both the count and the label inside one link.
- The "pending" count only shows when `current_user == @user && @user.private?`: you can only see your _own_ pending requests, and only a private account has any. `pending_path` takes no username, since `/pending` is always the current user's page.
- We use `@user.photos_count` (the counter cache column) instead of `@user.own_photos.count`, which saves a database query.

`flex-wrap` lets the counts wrap onto a second line on a narrow screen.

<div class="alert alert-success">

<i class="fa-solid fa-circle-check"></i> Visit `/alice`. You should see the display name, username, and stats, with clickable followers and following links.
</div>

### Bio and website

Add the user's bio and website link:

```erb{4-6,8}
<!-- ... -->
</div>

<p>
  <%= @user.bio %>
</p>

<%= link_to @user.website, @user.website, target: "_blank" %>
<!-- ... -->
```
{: filename="app/views/users/show.html.erb" }

`target: "_blank"` opens the website in a new browser tab.

### Posts and Likes tabs

Finally, add the Posts and Likes tabs to the bottom of the file:

```erb{4-18,20-29,31-34}
<!-- ... -->
<%= link_to @user.website, @user.website, target: "_blank" %>

<div data-controller="tabs" class="mt-6">
  <ul class="nav nav-underline mb-4" role="tablist">
    <li class="nav-item">
      <button type="button" class="nav-link active" data-tabs-target="tab" data-action="click->tabs#select" aria-controls="posts-pane" aria-selected="true">
        Posts
        <span class="badge text-bg-secondary"><%= @user.photos_count %></span>
      </button>
    </li>
    <li class="nav-item">
      <button type="button" class="nav-link" data-tabs-target="tab" data-action="click->tabs#select" aria-controls="likes-pane" aria-selected="false">
        Likes
        <span class="badge text-bg-secondary"><%= @user.likes_count %></span>
      </button>
    </li>
  </ul>

  <div id="posts-pane" class="tab-pane active" data-tabs-target="panel">
    <% if @user.photos_count.positive? %>
      <%= render @user.own_photos.pinned.latest %>
      <%= render @user.own_photos.unpinned.latest %>
    <% else %>
      <p class="text-zinc-500">
        User hasn't posted anything yet.
      </p>
    <% end %>
  </div>

  <div id="likes-pane" class="tab-pane" data-tabs-target="panel">
    <%= render @user.liked_photos.latest %>
  </div>
</div>
```
{: filename="app/views/users/show.html.erb" }

This is the `tabs` Stimulus controller from our starting point, used the same way as in [the component classes lesson](/lessons/912-appdev-stylesheet-component-classes#level-5-tabs):

- `data-controller="tabs"` goes on a `<div>` that holds both the tabs and the panels.
- Each tab is a button with `data-tabs-target="tab"` and `data-action="click->tabs#select"`. Its `aria-controls` names the `id` of the panel it shows.
- Each panel has `data-tabs-target="panel"`. The tab and the panel that start out visible have the class `active`.

The panels' `id`s are there because the page itself needs them: `aria-controls` points at them, both for the controller and for screen readers.

In the **Posts** panel, we render pinned photos first, and then the rest, with the `pinned` and `unpinned` scopes we defined on the Photo model in an earlier lesson. In the **Likes** panel, we render the photos the user has liked, through the `liked_photos` association. Each group is ordered newest first with the `latest` scope from the same lesson.

<div class="alert alert-success">

<i class="fa-solid fa-circle-check"></i> Visit `/alice`. You should see the whole profile page. Her pinned photo is at the top of Posts, with the "Pinned" badge. Click "Likes" to see the photos she's liked, and "Posts" to switch back.
</div>

Commit:

```
git add -A
git commit -m "Built user profile page with tabs"
git push
```

## Where we are

We're about halfway through this lesson. So far, we've built:

- The follow button, comment, and like partials.
- The photo card, which uses all three.
- The feed and discover pages, which render a photo card for each photo.
- The profile page, with its banner, stats, and Posts and Likes tabs.

Next come the pages that list _people_: followers, following, pending follow requests, search results, and the "Liked by" page, plus the photo's own page. Then we'll customize the Devise pages for signing in, signing up, and changing your settings.

## User list item partial

The followers, following, and search results pages all show lists of users. Let's build a partial for one user in a list before we tackle those pages.

Create `app/views/users/_list_item.html.erb`:

```erb
<li>
  <div class="flex gap-3">
    <%= image_tag user.avatar_image, class: "rounded-full img-small img-cover shrink-0" %>

    <div class="grow">
      <div class="flex items-center justify-between">
        <%= link_to user_path(user.username) do %>
          <%= user.display_name %>
          <span class="block font-light text-zinc-500">@<%= user.username %></span>
        <% end %>

        <%= render "follow_requests/follow_unfollow", sender: current_user, recipient: user %>
      </div>

      <p class="mb-0">
        <%= user.bio %>
      </p>
    </div>
  </div>
</li>
```
{: filename="app/views/users/_list_item.html.erb" copyable }

This has the same shape as the like partial: avatar on the left, then the user's name and username, a follow button on the right, and the bio below. The name links to the user's profile page. `block` puts the username on its own line.

## Followers page

Replace the stub in `app/views/users/followers.html.erb`:

```erb
<% content_for :title, "Followers" %>

<%= link_to :back, class: "icon-link mb-4" do %>
  <i class="fa-solid fa-arrow-left"></i>
  Back
<% end %>

<h1>Followers</h1>

<% if @followers.empty? %>
  <p class="text-zinc-500">
    User doesn't have any followers yet.
  </p>
<% else %>
  <div class="card">
    <header>
      <%= pluralize(@followers.count, "follower") %>
    </header>

    <ul>
      <% @followers.each do |user| %>
        <%= render "users/list_item", user: user %>
      <% end %>
    </ul>
  </div>
<% end %>
```
{: filename="app/views/users/followers.html.erb" copyable }

The list is a card with a `<header>` and a `<ul>`, which our stylesheet draws as [a list with dividers](/lessons/945-style-with-appdev-tailwind#flush-list-groups) that runs to the edges of the card. If the user has no followers, we show a message instead.

We render the partial in an `each` loop here. Collection rendering would work too, but the partial is named `list_item`, while the local variable it expects is `user`. The `as:` option names the variable: `render partial: "users/list_item", collection: @followers, as: :user`.

The Back link uses `link_to :back`. Instead of a path, `:back` tells Rails to link to the page the browser came from, which the browser sends with each request in a header called `Referer`. If the request has no `Referer` (say, if someone typed the URL into the address bar), Rails falls back to a link that runs the browser's own Back button, `javascript:history.back()`.

<div class="alert alert-success">

<i class="fa-solid fa-circle-check"></i> On `/alice`, click "followers." You should see Alice's followers, with avatars, usernames, and follow buttons. Click "Back" to return to her profile.
</div>

![Alice's followers page: a card listing each follower with their avatar, name, username, bio, and a follow button](/assets/followers.png)

## Following page

Replace the stub in `app/views/users/follows.html.erb`:

```erb
<% content_for :title, "Following" %>

<%= link_to :back, class: "icon-link mb-4" do %>
  <i class="fa-solid fa-arrow-left"></i>
  Back
<% end %>

<h1>Following</h1>

<% if @follows.empty? %>
  <p class="text-zinc-500">
    User hasn't followed anyone yet.
  </p>
<% else %>
  <div class="card">
    <header>
      <%= @follows.count %> following
    </header>

    <ul>
      <% @follows.each do |user| %>
        <%= render "users/list_item", user: user %>
      <% end %>
    </ul>
  </div>
<% end %>
```
{: filename="app/views/users/follows.html.erb" copyable }

This follows the same pattern as the followers page, but lists `@follows` (the user's leaders).

<div class="alert alert-success">

<i class="fa-solid fa-circle-check"></i> On `/alice`, click "following." You should see the people Alice follows.
</div>

Commit:

```
git add -A
git commit -m "Added user list item partial, followers, and following"
git push
```

## Pending page

Unlike the followers and following pages, the pending page needs Accept and Reject buttons. So instead of a list of users, it goes through the pending follow _requests_, and builds forms that update each one.

Replace the stub in `app/views/users/pending.html.erb`:

```erb{21-22,36-52}
<% content_for :title, "Pending" %>

<%= link_to :back, class: "icon-link mb-4" do %>
  <i class="fa-solid fa-arrow-left"></i>
  Back
<% end %>

<h1>Pending</h1>

<% if @pending.empty? %>
  <p class="text-zinc-500">
    User doesn't have any pending follows.
  </p>
<% else %>
  <div class="card">
    <header>
      <%= @pending.count %> pending
    </header>

    <ul>
      <% @pending.each do |follow_request| %>
        <% user = follow_request.sender %>

        <li>
          <div class="flex gap-3">
            <%= image_tag user.avatar_image, class: "rounded-full img-small img-cover shrink-0" %>

            <div class="grow">
              <div class="flex items-center justify-between">
                <%= link_to user_path(user.username) do %>
                  <%= user.display_name %>
                  <span class="block font-light text-zinc-500">@<%= user.username %></span>
                <% end %>

                <div class="flex gap-2">
                  <%= form_with(model: follow_request) do |form| %>
                    <%= form.hidden_field :status, value: "accepted" %>

                    <%= form.button class: "btn btn-primary rounded-full icon-link" do %>
                      <i class="fa-solid fa-check"></i>
                      Accept
                    <% end %>
                  <% end %>

                  <%= form_with(model: follow_request) do |form| %>
                    <%= form.hidden_field :status, value: "rejected" %>

                    <%= form.button class: "btn btn-outline-secondary rounded-full icon-link" do %>
                      <i class="fa-solid fa-xmark"></i>
                      Reject
                    <% end %>
                  <% end %>
                </div>
              </div>

              <p class="mb-0">
                <%= user.bio %>
              </p>
            </div>
          </div>
        </li>
      <% end %>
    </ul>
  </div>
<% end %>
```
{: filename="app/views/users/pending.html.erb" copyable }

Each follow request gets two forms, highlighted above. Both send a PATCH request to `FollowRequestsController#update`, because `form_with(model: follow_request)` is given a saved record, but they send different values in their hidden `status` field:

- The **Accept** form sends `status` `"accepted"`.
- The **Reject** form sends `status` `"rejected"`.

That's all `update` needs, since, as we set it up in the previous lesson, it only changes a follow request's `status`.

The rest of each `<li>` is the same as our `_list_item` partial, except for the two buttons in place of the follow button. We could give the partial a flag that decides which buttons to show, but then every page that renders it would need to know about the flag, and the partial would hold two partials' worth of logic. When a partial would have to behave two different ways, it's usually simpler to write the second version out.

<div class="alert alert-success">

<i class="fa-solid fa-circle-check"></i> Alice's account is public, so she has no pending requests. Sign out, and sign in as `bob@example.com` (password `appdev`), whose account is private. Click "pending" on his profile page, or visit `/pending`. You should see his pending follow requests, with Accept and Reject buttons. Try accepting or rejecting one.
</div>

![Bob's pending page: a card listing each pending follow request with Accept and Reject buttons](/assets/pending.png)

Commit:

```
git add -A
git commit -m "Added pending page"
git push
```

## Search results

The users index page shows search results. `UsersController#index` gets the matching users from Ransack, and the page reuses our `_list_item` partial.

Replace the stub in `app/views/users/index.html.erb`:

```erb
<% content_for :title, "Search results" %>

<h1>Search results</h1>

<div class="card">
  <header>
    <%= pluralize(@users.count, "user") %>
  </header>

  <ul>
    <% @users.each do |user| %>
      <%= render "users/list_item", user: user %>
    <% end %>
  </ul>
</div>
```
{: filename="app/views/users/index.html.erb" copyable }

The search form in the right sidebar submits to this page, and `@users` holds the results.

<div class="alert alert-success">

<i class="fa-solid fa-circle-check"></i> On a wide screen, search for part of a username in the right sidebar (try `a`). You should see the matching users, with their avatars and follow buttons.
</div>

![The search results page for the letter "a": a card listing the matching users](/assets/search-results.png)

## Photo show and edit pages

Let's replace the scaffold-generated photo show and edit pages.

Replace `app/views/photos/show.html.erb`:

```erb
<%= link_to :back, class: "icon-link mb-4" do %>
  <i class="fa-solid fa-arrow-left"></i>
  Back
<% end %>

<%= render @photo %>
```
{: filename="app/views/photos/show.html.erb" copyable }

`render @photo` renders a single record the way `render @photos` renders a collection: Rails finds `photos/_photo.html.erb` and passes it `photo: @photo`. The photo card holds all the details.

Replace `app/views/photos/edit.html.erb`:

```erb
<% content_for :title, "Editing photo" %>

<%= link_to :back, class: "icon-link mb-4" do %>
  <i class="fa-solid fa-arrow-left"></i>
  Back
<% end %>

<h1>Editing photo</h1>

<div class="card">
  <%= render "form", photo: @photo %>
</div>
```
{: filename="app/views/photos/edit.html.erb" copyable }

<div class="alert alert-success">

<i class="fa-solid fa-circle-check"></i> Visit your profile page, open the `...` menu on one of your photos, and click "Edit." You should see the edit form, with the current image above the file field.
</div>

The scaffold left two more pages in the same plain shape. `app/views/photos/new.html.erb` is where the phone navbar's "Add photo" link goes, since the modal's buttons aren't in that navbar. Replace it:

```erb
<% content_for :title, "New photo" %>

<h1>New photo</h1>

<div class="card">
  <%= render "form", photo: @photo %>
</div>
```
{: filename="app/views/photos/new.html.erb" copyable }

And `app/views/comments/edit.html.erb` is where the "Edit" option on a comment goes. The scaffold's version ends with links to a page for one comment and to the list of every comment, which nothing else in the app links to, so those two links go. Replace it:

```erb
<% content_for :title, "Editing comment" %>

<%= link_to :back, class: "icon-link mb-4" do %>
  <i class="fa-solid fa-arrow-left"></i>
  Back
<% end %>

<h1>Editing comment</h1>

<div class="card">
  <%= render "form", comment: @comment %>
</div>
```
{: filename="app/views/comments/edit.html.erb" copyable }

<div class="alert alert-success">

<i class="fa-solid fa-circle-check"></i> Narrow your browser window and click "Add photo" in the top navbar. You should see the new photo form in a card. Then, on any photo, open the `...` menu on a comment and click "Edit." The comment's edit page should look like the photo's.
</div>

## Liked by page

When someone clicks the number of likes on a photo, they see everyone who liked it. This uses the nested route `/photos/:photo_id/likes`, which goes to `LikesController#index`.

Replace the scaffold-generated `app/views/likes/index.html.erb`:

```erb
<% content_for :title, "Liked by" %>

<%= link_to @photo, class: "icon-link mb-4" do %>
  <i class="fa-solid fa-arrow-left"></i>
  Back
<% end %>

<h1>Liked by</h1>

<div class="card">
  <header>
    <%= pluralize(@photo.likes_count, "like") %>
  </header>

  <ul>
    <%= render @likes %>
  </ul>
</div>
```
{: filename="app/views/likes/index.html.erb" copyable }

`render @likes` renders our `likes/_like.html.erb` partial once for each like.

This page's Back link goes to the photo itself, rather than `:back`. Each person in the list has a follow button, and after a follow, `redirect_back` brings you back to this same page. Then the "page you came from" is the "Liked by" page itself, and `:back` would go nowhere. When we know where Back should go, a real path is more predictable.

Here's a check on `link_to :back`.

- A page has `<%= link_to "Back", :back %>`. Where does the link go?
- To the page the browser says it came from, in the `Referer` header
  - Yes! And if there's no `Referer`, Rails uses `javascript:history.back()`.
- Always to the root page
  - Not quite. `:back` doesn't name any route of ours.
- To the previous route in `config/routes.rb`
  - Not quite. Rails reads where the browser came from, not the order of the routes.
{: .choose_best #link_to_back title="Linking back" points="1" answer="1" }

<div class="alert alert-success">

<i class="fa-solid fa-circle-check"></i> Click the number of likes on any photo. You should see everyone who liked it, each with a follow button. Click "Back" to return to the photo.
</div>

![The Liked by page for a photo, signed in as Alice: a card listing the two people who liked it, Alice herself and Bob, with a Following button beside Bob](/assets/liked-by.png)

Commit:

```
git add -A
git commit -m "Added search results, photo show and edit, and likes pages"
git push
```

## Customizing Devise views

Every page is built. Let's see what the tests think. Run `rspec`:

```
rspec
```

You should see `66 examples, 42 failures`. That's as many as at the end of the previous lesson! Scroll up to the first failure:

```
  1) /discover can be visited
     Failure/Error: click_button "Sign in"

     Capybara::ElementNotFound:
       Unable to find button "Sign in" that is not disabled
```

Almost every failure is this same one. The tests sign in with a helper method at the bottom of each feature spec file (this failure's is `spec/features/discover_spec.rb`):

```rb{6}
def sign_in(user)
  visit "/users/sign_in"

  fill_in "Email", with: user.email
  fill_in "Password", with: user.password
  click_button "Sign in"
end
```

The test clicks a button called "Sign in," but Devise's sign-in page says "Log in." One small difference in one shared helper fails 42 tests, which is why it's worth reading the first failure carefully before trying to fix all of them.

To change Devise's pages, we need copies of its view templates in our project. You did this in [Bulletin Board, Part 3](/lessons/726-bulletin-board-accounts#usernames-in-the-devise-forms). This time, we'll put the copies in a folder named after our users, and only copy the two folders of views we need. Run:

```
rails g devise:views users -v registrations sessions
```
{: copyable }

- `users` is our Devise scope (from `devise_for :users` in the routes), and puts the views in `app/views/users/` instead of `app/views/devise/`. That keeps them next to our other user pages.
- `-v registrations sessions` only copies the sign-up and settings views (`registrations`) and the sign-in view (`sessions`), instead of every page Devise has. The generator also adds the `shared` partials that those views render.

### Sign in view

Replace the generated `app/views/users/sessions/new.html.erb`:

```erb
<% content_for :title, "Sign in" %>

<h1>Sign in</h1>

<div class="card">
  <%= form_for(resource, as: resource_name, url: session_path(resource_name)) do |f| %>
    <%= render "users/shared/error_messages", resource: resource %>

    <div class="form-group">
      <%= f.label :email %>
      <%= f.email_field :email, autofocus: true, autocomplete: "email", class: "form-control" %>
    </div>

    <div class="form-group">
      <%= f.label :password %>
      <%= f.password_field :password, autocomplete: "current-password", class: "form-control" %>
    </div>

    <% if devise_mapping.rememberable? %>
      <div class="form-check form-group">
        <%= f.check_box :remember_me, class: "form-check-input" %>
        <%= f.label :remember_me, class: "form-check-label" %>
      </div>
    <% end %>

    <%= f.submit "Sign in", class: "btn btn-primary" %>
  <% end %>
</div>

<%= render "users/shared/links" %>
```
{: filename="app/views/users/sessions/new.html.erb" copyable }

The changes from the generated file: "Log in" is now "Sign in," in the heading and on the button; the form is in a card; and each field is a `form-group`, with `form-control` on the inputs and `form-check` classes on the "Remember me" checkbox, as in [the AppDev styles reference](/lessons/945-style-with-appdev-tailwind#forms).

Sign out, and visit `/users/sign_in`.

It still says "Log in"!

By default, Devise only looks for its views in `app/views/devise/`, which is where they went in Bulletin Board. To make it look in `app/views/users/`, open `config/initializers/devise.rb` and find the `scoped_views` setting, which is commented out:

```ruby{3}
  # ...
  # config.scoped_views = false
  config.scoped_views = true
  # ...
```
{: filename="config/initializers/devise.rb" }

Files in `config/initializers/` are only read when the server starts, so stop your server (<kbd>Ctrl</kbd> + <kbd>C</kbd>) and start it again with `bin/dev`.

<div class="alert alert-success">

<i class="fa-solid fa-circle-check"></i> Visit `/users/sign_in`. You should see a "Sign in" heading and button, and the form in a card.
</div>

![The sign-in page: a card with Email and Password fields, a Remember me checkbox, and a Sign in button](/assets/sign-in.png)

Run `rspec` again. You should be down to `66 examples, 1 failure`:

```
  1) User authentication allows new user sign ups
     Failure/Error: fill_in "Username", with: "alice"

     Capybara::ElementNotFound:
       Unable to find field "Username" that is not disabled
```

Commit:

```
git add -A
git commit -m "Generated Devise views and customized sign in"
git push
```

### Sign up view

Every user needs a username, but Devise's sign-up form doesn't have a field for it. Let's add one, and one for the display name. First, the generated error messages partial, which the sign-up and settings forms render when a save fails, gets our alert styling. Replace `app/views/users/shared/_error_messages.html.erb`:

```erb
<% if resource.errors.any? %>
  <div class="alert alert-danger" role="alert">
    <strong>
      <%= I18n.t("errors.messages.not_saved",
                 count: resource.errors.count,
                 resource: resource.class.model_name.human.downcase)
       %>
    </strong>

    <ul class="mb-0">
      <% resource.errors.full_messages.each do |message| %>
        <li><%= message %></li>
      <% end %>
    </ul>
  </div>
<% end %>
```
{: filename="app/views/users/shared/_error_messages.html.erb" copyable }

Then replace the generated `app/views/users/registrations/new.html.erb`:

```erb{14-17,19-22}
<% content_for :title, "Sign up" %>

<h1>Sign up</h1>

<div class="card">
  <%= form_for(resource, as: resource_name, url: registration_path(resource_name)) do |f| %>
    <%= render "users/shared/error_messages", resource: resource %>

    <div class="form-group">
      <%= f.label :email %>
      <%= f.email_field :email, autofocus: true, autocomplete: "email", class: "form-control" %>
    </div>

    <div class="form-group">
      <%= f.label :display_name %>
      <%= f.text_field :display_name, class: "form-control" %>
    </div>

    <div class="form-group">
      <%= f.label :username %>
      <%= f.text_field :username, class: "form-control" %>
    </div>

    <div class="form-group">
      <%= f.label :password %>
      <%= f.password_field :password, autocomplete: "new-password", class: "form-control" %>

      <% if @minimum_password_length %>
        <small class="form-text">
          <%= @minimum_password_length %> characters minimum.
        </small>
      <% end %>
    </div>

    <div class="form-group">
      <%= f.label :password_confirmation %>
      <%= f.password_field :password_confirmation, autocomplete: "new-password", class: "form-control" %>
    </div>

    <%= f.submit "Sign up", class: "btn btn-primary" %>
  <% end %>
</div>

<%= render "users/shared/links" %>
```
{: filename="app/views/users/registrations/new.html.erb" copyable }

The highlighted lines are the two new fields, between the email and the password. Sign out, visit `/users/sign_up`, and sign up as a new user, with a display name and a username.

You'll see "Username can't be blank," even though you filled it in!

Run `rspec`, and the sign-up test fails the same way:

```
  1) User authentication allows new user sign ups
     Failure/Error: expect(page).to have_content("Welcome! You have signed up successfully")
       expected to find text "Welcome! You have signed up successfully" in "...Username can't be blank..."
```

You've seen this one before, in [Bulletin Board, Part 3](/lessons/726-bulletin-board-accounts#permit-the-new-field): Devise ignores every form field it hasn't been told to accept. By default, it accepts the email and the two passwords, and nothing else.

### Permitted parameters

Open `app/controllers/application_controller.rb` and add the highlighted lines:

```ruby{2,10,12-15}
class ApplicationController < ActionController::Base
  before_action :configure_permitted_parameters, if: :devise_controller?
  before_action :authenticate_user!
  before_action :set_user_search, if: -> { current_user.present? }

  def set_user_search
    @q = User.all.ransack(params[:q])
  end

  protected

  def configure_permitted_parameters
    devise_parameter_sanitizer.permit(:sign_up, keys: [ :display_name, :username ])
    devise_parameter_sanitizer.permit(:account_update, keys: [ :avatar_image, :bio, :display_name, :username, :private, :profile_banner, :remove_profile_banner, :website ])
  end
end
```
{: filename="app/controllers/application_controller.rb" }

- `if: :devise_controller?` means the `before_action` only runs for Devise's own controllers (sign up, sign in, settings), not for ours.
- `:sign_up` lists the extra fields that the sign-up form may send: the display name and the username. A new user doesn't need an avatar or a bio yet.
- `:account_update` lists the extra fields for the settings form, which we'll build next. `:remove_profile_banner` is the virtual attribute we added to the User model in an earlier lesson, for a checkbox that removes the banner.

`protected` works like `private` here: Devise can call `configure_permitted_parameters` as a `before_action`, but it isn't an action that a route could reach.

Here's a check on permitted parameters.

- In a recipe app with Devise, you add a `phone` column to `users` and a phone field to the sign-up form. After signing up, the new user's `phone` is blank. What's missing?
- A `validates :phone, presence: true` in the User model
  - Not quite. A validation would reject the blank phone, not save the one that was typed.
- `:phone` in the `devise_parameter_sanitizer.permit(:sign_up, ...)` keys
  - Yes! Devise drops any field it hasn't been told to accept.
- `:phone` in the `:account_update` keys
  - Not quite. That list is for the settings form, not the sign-up form.
{: .choose_best #permitted_params title="Permitted parameters" points="1" answer="2" }

<div class="alert alert-success">

<i class="fa-solid fa-circle-check"></i> Visit `/users/sign_up` again, and sign up with a display name and a username. You should be signed in, with your username in the left sidebar.
</div>

![The sign-up page: a card with Email, Display name, Username, Password, and Password confirmation fields](/assets/sign-up.png)

Commit:

```
git add -A
git commit -m "Customized sign up and permitted parameters"
git push
```

### Settings view

The settings page (`/users/edit`, "Settings" in the sidebar) is the biggest Devise form. It has fields for the password, the profile (username, display name, avatar, banner, bio, website), and the private account checkbox.

Replace the generated `app/views/users/registrations/edit.html.erb`. We'll write the start of it together, so delete what's in it now:

```erb
<% content_for :title, "Settings" %>

<h1>Settings</h1>

<div class="card">
  <%= form_for(resource, as: resource_name, url: registration_path(resource_name), html: { method: :put, novalidate: true }) do |f| %>
    <%= render "users/shared/error_messages", resource: resource %>

    <% was_validated = resource.errors.any? %>

    <div class="form-group">
      <% current_password_was_invalid = resource.errors.include?(:current_password) %>

      <% current_password_class = "form-control" %>

      <% if was_validated %>
        <% if current_password_was_invalid %>
          <% current_password_class << " is-invalid" %>
        <% else %>
          <% current_password_class << " is-valid" %>
        <% end %>
      <% end %>

      <%= f.label :current_password %>

      <%= f.password_field :current_password, class: current_password_class, autocomplete: "current-password" %>

      <% resource.errors.full_messages_for(:current_password).each do |message| %>
        <div class="invalid-feedback">
          <%= message %>
        </div>
      <% end %>

      <small class="form-text">
        We need your current password to confirm any changes.
      </small>
    </div>

    <hr>
<!-- ... -->
```
{: filename="app/views/users/registrations/edit.html.erb" }

Devise asks for the current password before it saves any change, so that someone who finds your laptop signed in can't take over your account. The form tag's `novalidate: true` turns off the browser's own checks and pop-ups, so that every error comes from our server, and appears in the same place.

The current password field also shows the user _which_ field went wrong. After a failed save, `resource.errors.any?` is `true`, so `was_validated` is `true`, and:

1. `resource.errors.include?(:current_password)` checks whether this field has an error.
2. We start with the class `"form-control"`, and add `" is-invalid"` (a red border) if it had an error, or `" is-valid"` (a green border) if it didn't. `<<` adds the text to the end of the string.
3. We render the field with that class.
4. We show each of the field's error messages in a `div.invalid-feedback`, in red under the field.

Now write the rest of the form yourself, using what you've seen in the sign-up form. Each field is a `form-group` with a label and an input with the class `form-control`:

- The email, password, and password confirmation fields. Under the password, add a `form-text` hint: "Leave blank if you don't want to change it." Then another `<hr>`.
- Text fields for the username and display name.
- The avatar, with a `file_field` (and `accept: "image/*"`, so the file picker only shows images), and the current avatar above it: `image_tag current_user.avatar_image, class: "rounded-full img-cover img-large mb-2"`.
- The profile banner, the same way, but only show the current banner if one is attached. Below the file field, if a banner is attached, add a checkbox for `:remove_profile_banner` with the label "Remove profile banner."
- A `text_area` for the bio, and a `url_field` for the website.
- A checkbox for `:private` with the label "Private account."
- An "Update" submit button, and the closing tags.

For each checkbox, use the `form-check` shape from the "Remember me" checkbox in the sign-in form.

<div class="alert alert-success">

<i class="fa-solid fa-circle-check"></i> Visit `/users/edit` (or click "Settings" in the sidebar). You should see the whole settings form. Change your bio, enter your current password, and click "Update." Then try again without your current password: the field should turn red, with an error message under it.
</div>

![The settings page, signed in as Alice: a card with the current password field, then the email, password, username, and display name fields, and the top of her current avatar](/assets/settings.png)

Compare your form with the whole file, and fix any differences before you continue:

<button class="btn btn-primary" type="button" data-bs-toggle="collapse" data-bs-target="#settingsForm" aria-expanded="false" aria-controls="settingsForm">
Click to see the whole settings form
</button>
<div class="collapse" id="settingsForm">
  <div class="card card-body">

```erb
<% content_for :title, "Settings" %>

<h1>Settings</h1>

<div class="card">
  <%= form_for(resource, as: resource_name, url: registration_path(resource_name), html: { method: :put, novalidate: true }) do |f| %>
    <%= render "users/shared/error_messages", resource: resource %>

    <% was_validated = resource.errors.any? %>

    <div class="form-group">
      <% current_password_was_invalid = resource.errors.include?(:current_password) %>

      <% current_password_class = "form-control" %>

      <% if was_validated %>
        <% if current_password_was_invalid %>
          <% current_password_class << " is-invalid" %>
        <% else %>
          <% current_password_class << " is-valid" %>
        <% end %>
      <% end %>

      <%= f.label :current_password %>

      <%= f.password_field :current_password, class: current_password_class, autocomplete: "current-password" %>

      <% resource.errors.full_messages_for(:current_password).each do |message| %>
        <div class="invalid-feedback">
          <%= message %>
        </div>
      <% end %>

      <small class="form-text">
        We need your current password to confirm any changes.
      </small>
    </div>

    <hr>

    <div class="form-group">
      <%= f.label :email %>
      <%= f.email_field :email, class: "form-control", autocomplete: "email" %>
    </div>

    <div class="form-group">
      <%= f.label :password %>
      <%= f.password_field :password, class: "form-control", autocomplete: "new-password" %>

      <small class="form-text">
        Leave blank if you don't want to change it.
        <% if @minimum_password_length %>
          <%= @minimum_password_length %> characters minimum.
        <% end %>
      </small>
    </div>

    <div class="form-group">
      <%= f.label :password_confirmation %>
      <%= f.password_field :password_confirmation, class: "form-control", autocomplete: "new-password" %>
    </div>

    <hr>

    <div class="form-group">
      <%= f.label :username %>
      <%= f.text_field :username, class: "form-control" %>
    </div>

    <div class="form-group">
      <%= f.label :display_name %>
      <%= f.text_field :display_name, class: "form-control" %>
    </div>

    <div class="form-group">
      <%= f.label :avatar_image %>
      <%= image_tag current_user.avatar_image, class: "rounded-full img-cover img-large mb-2" %>
      <%= f.file_field :avatar_image, class: "form-control", accept: "image/*" %>
    </div>

    <div class="form-group">
      <%= f.label :profile_banner %>

      <% if current_user.profile_banner.attached? %>
        <%= image_tag current_user.profile_banner, class: "w-full rounded-lg mb-2" %>
      <% end %>

      <%= f.file_field :profile_banner, class: "form-control", accept: "image/*" %>

      <% if current_user.profile_banner.attached? %>
        <div class="form-check mt-2">
          <%= f.check_box :remove_profile_banner, class: "form-check-input" %>
          <%= f.label :remove_profile_banner, "Remove profile banner", class: "form-check-label" %>
        </div>
      <% end %>
    </div>

    <div class="form-group">
      <%= f.label :bio %>
      <%= f.text_area :bio, class: "form-control" %>
    </div>

    <div class="form-group">
      <%= f.label :website %>
      <%= f.url_field :website, class: "form-control" %>
    </div>

    <div class="form-check form-group">
      <%= f.check_box :private, class: "form-check-input" %>
      <%= f.label :private, "Private account", class: "form-check-label" %>
    </div>

    <%= f.submit "Update", class: "btn btn-primary" %>
  <% end %>
</div>
```
{: filename="app/views/users/registrations/edit.html.erb" copyable }

  </div>
</div>

Commit:

```
git add -A
git commit -m "Built the settings form"
git push
```

## Finish it off

Run `rspec` one more time.

<div class="alert alert-success">

<i class="fa-solid fa-circle-check"></i> You should see `66 examples, 0 failures`. Every page of the app is built. Run `grade` to record your full points for the Photogram Industrial project, and then make a final commit and push.
</div>

## Wrapping up

Our Photogram is now a complete app: a feed, a discover page, profiles with tabs, follower lists, pending requests, search, and Devise pages that match the rest of the app. Every page is built from a handful of partials (the photo card, the follow button, the list item), and most pages are only a few lines long because of them.

In the next lesson, we'll make sure people can only do what they're allowed to do: edit their own photos, see a private account's photos only once they're following it, and so on.

---

- Approximately how long (in minutes) did this lesson take you to complete?
{: .free_text_number #time_taken title="Time taken" points="1" answer="any" }

<!--

# List of project specs for AI assistant

require "rails_helper"

RSpec.describe Comment, type: :model do
  describe "has a belongs_to association defined called 'author' with Class name 'User'", points: 1 do
    it { should belong_to(:author).class_name("User") }
  end

  describe "has a belongs_to association defined called 'photo'", points: 1 do
    it { should belong_to(:photo) }
  end
end

require "rails_helper"

RSpec.describe FollowRequest, type: :model do
  describe "has a belongs_to association defined called 'sender' with Class name 'User'", points: 1 do
    it { should belong_to(:sender).class_name("User") }
  end

  describe "has a belongs_to association defined called 'recipient' with Class name 'User'", points: 1 do
    it { should belong_to(:recipient).class_name("User") }
  end
end

require "rails_helper"

RSpec.describe Like, type: :model do
  describe "has a belongs_to association defined called 'fan' with Class name 'User'", points: 1 do
    it { should belong_to(:fan).class_name("User") }
  end
end

RSpec.describe Like, type: :model do
  describe "has a belongs_to association defined called 'photo'", points: 1 do
    it { should belong_to(:photo) }
  end
end

require "rails_helper"

RSpec.describe Photo, type: :model do
  describe "has a belongs_to association defined called 'owner' with Class name 'User'", points: 1 do
    it { should belong_to(:owner).class_name("User") }
  end

  describe "has a has_many association defined called 'comments'", points: 1 do
    it { should have_many(:comments) }
  end

  describe "has a has_many association defined called 'likes'", points: 1 do
    it { should have_many(:likes) }
  end

  describe "has a has_many (many-to_many) association defined called 'fans' through 'likes'", points: 1 do
    it { should have_many(:fans).through(:likes) }
  end
end

require "rails_helper"


RSpec.describe User, type: :model do
  describe "has a has_many association defined called 'comments' with Class name 'Comment' and foreign key 'author_id'", points: 1 do
    it { should have_many(:comments).class_name("Comment").with_foreign_key("author_id") }
  end

  describe "has a has_many association defined called 'own_photos' with Class name 'Photo' and foreign key 'owner_id'", points: 1 do
    it { should have_many(:own_photos).class_name("Photo").with_foreign_key("owner_id") }
  end

  describe "has a has_many association defined called 'likes' with Class name 'Like' and foreign key 'fan_id'", points: 1 do
    it { should have_many(:likes).class_name("Like").with_foreign_key("fan_id") }
  end

  describe "has a has_many (many-to_many) association defined called 'liked_photos' through 'likes' and source 'photo'", points: 1 do
    it { should have_many(:liked_photos).through(:likes).source(:photo) }
  end

  describe "has a has_many association defined called 'sent_follow_requests' with Class name 'FollowRequest' and foreign key 'sender_id'", points: 1 do
    it { should have_many(:sent_follow_requests).class_name("FollowRequest").with_foreign_key("sender_id") }
  end

  describe "has a has_many association defined called 'received_follow_requests' with Class name 'FollowRequest' and foreign key 'recipient_id'", points: 1 do
    it { should have_many(:received_follow_requests).class_name("FollowRequest").with_foreign_key("recipient_id") }
  end

  describe "has a has_many association defined called 'accepted_sent_follow_requests' with scope where 'status' is \"accepted\"", points: 1 do
    it { should have_many(:accepted_sent_follow_requests).class_name("FollowRequest").with_foreign_key("sender_id").conditions(status: "accepted") }
  end

  describe "has a has_many association defined called 'accepted_received_follow_requests' with scope where 'status' is \"accepted\"", points: 1 do
    it { should have_many(:accepted_received_follow_requests).class_name("FollowRequest").with_foreign_key("recipient_id").conditions(status: "accepted") }
  end

  describe "has a has_many (many-to_many) association defined called 'followers' through 'accepted_received_follow_requests' and source 'sender'", points: 1 do
    it { should have_many(:followers).through(:accepted_received_follow_requests).source(:sender) }
  end

  describe "has a has_many (many-to_many) association defined called 'leaders' through 'accepted_sent_follow_requests' and source 'recipient'", points: 1 do
    it { should have_many(:leaders).through(:accepted_sent_follow_requests).source(:recipient) }
  end

  describe "has a has_many (many-to_many) association defined called 'feed' through 'leaders' and source 'own_photos'", points: 1 do
    it { should have_many(:feed).through(:leaders).source(:own_photos) }
  end

  describe "has a has_many (many-to_many) association defined called 'discover' through 'leaders' and source 'liked_photos'", points: 1 do
    it { should have_many(:discover).through(:leaders).source(:liked_photos) }
  end
end

require "rails_helper"

describe "User authentication" do
  it "displays a banner to sign in when trying to visit the homepage", points: 1 do
    visit "/"

    expect(page).to have_content("You need to sign in or sign up before continuing")
  end

  it "sends the user to the sign in page when trying to visit the homepage", points: 1 do
    visit "/"

    expect(page).to have_current_path("/users/sign_in")
  end

  it "allows new user sign ups", points: 1 do
    visit "/users/sign_up"

    fill_in "Email", with: "alice@example.com"
    fill_in "Password", with: "appdev"
    fill_in "Password confirmation", with: "appdev"
    fill_in "Username", with: "alice"
    click_button "Sign up"

    expect(page).to have_content("Welcome! You have signed up successfully")
  end

  it "allows an existing user to sign in", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")

    visit "/users/sign_in"

    fill_in "Email", with: user.email
    fill_in "Password", with: user.password
    click_button "Sign in"

    expect(page).to have_content("Signed in successfully")
  end

  it "allows a user to sign out", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")

    visit "/users/sign_in"

    fill_in "Email", with: user.email
    fill_in "Password", with: user.password
    click_button "Sign in"

    click_on user.username
    click_on "Sign out"

    expect(page).to have_current_path("/users/sign_in")
  end
end

require "rails_helper"

describe "/" do
  it "can be visited", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    visit "/"

    expect(page.status_code).to be(200)
  end

  it "has a navbar", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    visit "/"

    expect(page).to have_tag("nav", with: { class: "navbar" })
  end

  it "has a Settings link for the signed in user", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    visit "/"

    expect(page).to have_link("Settings", href: "/users/edit")
  end

  it "does not have a sign in link if the user is already signed in", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    visit "/"

    expect(page).to_not have_link("Sign in", href: "/users/sign_in")
  end

  it "has a link, 'Feed', that navigates to the 'Feed' page", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    visit "/"

    click_on "Feed"

    expect(page).to have_current_path("/feed")
  end

  it "has a link, 'Discover', that navigates to the 'Discover' page", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    visit "/"

    click_on "Discover"

    expect(page).to have_current_path("/discover")
  end

  it "has a link, 'Go to profile', that navigates to the profile page", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    visit "/"

    click_on "Go to profile"

    expect(page).to have_current_path("/#{user.username}")
  end

  it "has an 'Add photo' button", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    visit "/"

    expect(page).to have_button("Add photo")
  end
end

def sign_in(user)
  visit "/users/sign_in"

  fill_in "Email", with: user.email
  fill_in "Password", with: user.password
  click_button "Sign in"
end

require "rails_helper"

describe "/photos/new" do
  it "has a form to add a new photo", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    visit "/photos/new"

    expect(page).to have_form("/photos", :post)
  end

  it "does not allow the user to add a new photo without a caption", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    visit "/"

    click_button "Add photo"

    attach_file "Image", "#{Rails.root}/spec/support/test_image.jpeg"
    click_on "Create Photo"

    expect(page).to have_content("Caption can't be blank")
  end

  it "allows the user to add a new photo", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    visit "/"

    click_button "Add photo"

    attach_file "Image", "#{Rails.root}/spec/support/test_image.jpeg"
    fill_in "Caption", with: "caption"
    click_on "Create Photo"

    expect(page).to have_content("Photo was successfully created")
  end

  it "redirects to the photo details page after creating a new photo", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    visit "/"

    click_button "Add photo"

    attach_file "Image", "#{Rails.root}/spec/support/test_image.jpeg"
    fill_in "Caption", with: "caption"
    click_on "Create Photo"

    expect(page).to have_current_path("/photos/#{Photo.last.id}")
  end
end

describe "/photos/[ID]" do
  it "displays the photo and caption", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    photo = create_photo(owner: user, caption: "caption")

    visit "/photos/#{photo.id}"

    expect(page).to have_css("img")
    expect(page).to have_content(photo.caption)
  end

  it "allows the user to edit the photo", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    photo = create_photo(owner: user, caption: "caption")

    visit "/photos/#{photo.id}"

    click_on "Edit"

    all("textarea").last.fill_in(with: "new caption")
    all("input[type='submit']").last.click

    expect(page).to have_content("new caption")
  end
end

def sign_in(user)
  visit "/users/sign_in"

  fill_in "Email", with: user.email
  fill_in "Password", with: user.password
  click_button "Sign in"
end

def create_photo(owner:, caption: "caption")
  photo = Photo.new(caption: caption, owner_id: owner.id)
  photo.image.attach(io: File.open(Rails.root.join("spec/support/test_image.jpeg")), filename: "test_image.jpeg", content_type: "image/jpeg")
  photo.save!
  photo
end

require "rails_helper"

describe "/[USERNAME]" do
  it "can be visited", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    visit "/#{user.username}"

    expect(page.status_code).to be(200)
  end

  it "has a Posts tab", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    visit "/#{user.username}"

    expect(page).to have_button("Posts")
  end

  it "has a Likes tab", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    visit "/#{user.username}"

    expect(page).to have_button("Likes")
  end

  it "displays each of the user's photos", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    photo = create_photo(owner: user, caption: "caption")

    visit "/#{user.username}"

    expect(page).to have_css("img")
    expect(page).to have_content(photo.caption)
  end

  it "shows the comments on the user's photos", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    photo = create_photo(owner: user, caption: "caption")
    comment = Comment.create(body: "comment body", author_id: user.id, photo_id: photo.id)

    visit "/#{user.username}"

    expect(page).to have_content(comment.body)
  end

  it "allows the user to delete their photo", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    photo = create_photo(owner: user, caption: "caption")

    visit "/#{user.username}"

    click_on "Delete"

    expect(page).not_to have_content(photo.caption)
  end

  it "shows a list of followers on the user profile", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    other_user = User.create(username: "other_user", email: "other_user@example.com", password: "appdev")
    FollowRequest.create(sender_id: other_user.id, recipient_id: user.id, status: "accepted")

    visit "/#{user.username}"

    click_on "followers"

    expect(page).to have_content(other_user.username)
  end

  it "shows a list of leaders on the user profile", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    other_user = User.create(username: "other_user", email: "other_user@example.com", password: "appdev")
    FollowRequest.create(sender_id: user.id, recipient_id: other_user.id, status: "accepted")

    visit "/#{user.username}"

    click_on "following"

    expect(page).to have_content(other_user.username)
  end

  it "shows a 'Following' button for leaders", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    other_user = User.create(username: "other_user", email: "other_user@example.com", password: "appdev")
    FollowRequest.create(sender_id: user.id, recipient_id: other_user.id, status: "accepted")

    visit "/#{other_user.username}"

    expect(page).to have_button("Following")
  end

  it "shows pending follow requests for private accounts", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    private_user = User.create(username: "private_user", email: "private_user@example.com", password: "appdev", private: true)

    visit "/#{private_user.username}"

    click_on "Follow"

    expect(page).to have_button("Requested")
  end

  it "allows a user to unfollow another user", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    other_user = User.create(username: "other_user", email: "other_user@example.com", password: "appdev")
    FollowRequest.create(sender_id: user.id, recipient_id: other_user.id, status: "accepted")

    visit "/#{other_user.username}"

    click_on "Following"

    expect(page).to have_button("Follow")
  end

  it "allows a user to cancel pending follow request", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    private_user = User.create(username: "private_user", email: "private_user@example.com", password: "appdev", private: true)
    FollowRequest.create(sender_id: user.id, recipient_id: private_user.id, status: "pending")

    visit "/#{private_user.username}"

    click_on "Requested"

    expect(page).to have_button("Follow")
  end

  it "keeps a follow request to a private user pending, even if the form says otherwise", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    private_user = User.create(username: "bob", email: "bob@example.com", password: "appdev", private: true)

    page.driver.submit :post, "/follow_requests", { follow_request: { recipient_id: private_user.id, status: "accepted" } }

    expect(FollowRequest.find_by(sender_id: user.id, recipient_id: private_user.id).status).to eq("pending")
  end
end

describe "/pending" do
  it "allows a user to accept a follow request", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    other_user = User.create(username: "other_user", email: "other_user@example.com", password: "appdev")
    FollowRequest.create(sender_id: other_user.id, recipient_id: user.id, status: "pending")

    visit "/pending"

    click_on "Accept"

    expect(page).not_to have_content(other_user.username)
  end

  it "allows a user to reject a follow request", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    other_user = User.create(username: "other_user", email: "other_user@example.com", password: "appdev")
    FollowRequest.create(sender_id: other_user.id, recipient_id: user.id, status: "pending")

    visit "/pending"

    click_on "Reject"

    expect(page).not_to have_content(other_user.username)
  end
end

def sign_in(user)
  visit "/users/sign_in"

  fill_in "Email", with: user.email
  fill_in "Password", with: user.password
  click_button "Sign in"
end

def create_photo(owner:, caption: "caption")
  photo = Photo.new(caption: caption, owner_id: owner.id)
  photo.image.attach(io: File.open(Rails.root.join("spec/support/test_image.jpeg")), filename: "test_image.jpeg", content_type: "image/jpeg")
  photo.save!
  photo
end

require "rails_helper"

describe "/discover" do
  it "can be visited", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    visit "/discover"

    expect(page.status_code).to be(200)
  end

  it "shows photos liked by people the current user follows", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    leader = User.create(username: "leader", email: "leader@example.com", password: "appdev")
    owner = User.create(username: "owner", email: "owner@example.com", password: "appdev", private: false)
    photo = create_photo(owner: owner, caption: "owner caption")
    FollowRequest.create(sender_id: user.id, recipient_id: leader.id, status: "accepted")
    Like.create(fan_id: leader.id, photo_id: photo.id)

    visit "/discover"

    expect(page).to have_content(photo.caption)
  end
end

def sign_in(user)
  visit "/users/sign_in"

  fill_in "Email", with: user.email
  fill_in "Password", with: user.password
  click_button "Sign in"
end

def create_photo(owner:, caption: "caption")
  photo = Photo.new(caption: caption, owner_id: owner.id)
  photo.image.attach(io: File.open(Rails.root.join("spec/support/test_image.jpeg")), filename: "test_image.jpeg", content_type: "image/jpeg")
  photo.save!
  photo
end

require "rails_helper"

describe "/feed" do
  it "can be visited", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    visit "/feed"

    expect(page.status_code).to be(200)
  end

  it "shows their leader's photos", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    leader = User.create(username: "leader", email: "leader@example.com", password: "appdev", private: false)
    photo = create_photo(owner: leader, caption: "leader caption")
    FollowRequest.create(sender_id: user.id, recipient_id: leader.id, status: "accepted")

    visit "/feed"

    expect(page).to have_content(photo.caption)
    expect(page).to have_tag("img")
  end

  it "allows them to like their leader's photos", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    leader = User.create(username: "leader", email: "leader@example.com", password: "appdev", private: false)
    photo = create_photo(owner: leader)
    FollowRequest.create(sender_id: user.id, recipient_id: leader.id, status: "accepted")

    visit "/feed"

    click_on "0 likes"

    expect(page).to have_tag("i", with: { class: [ "fa-solid", "fa-heart" ] })
  end

  it "allows them to un-like their leader's photos", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    leader = User.create(username: "leader", email: "leader@example.com", password: "appdev", private: false)
    photo = create_photo(owner: leader)
    FollowRequest.create(sender_id: user.id, recipient_id: leader.id, status: "accepted")
    Like.create(fan_id: user.id, photo_id: photo.id)

    visit "/feed"

    click_on "1 like"

    expect(page).to have_css("i.fa-regular.fa-heart")
  end

  it "allows the user to add a comment on their leader's photos", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    leader = User.create(username: "leader", email: "leader@example.com", password: "appdev", private: false)
    photo = create_photo(owner: leader)
    FollowRequest.create(sender_id: user.id, recipient_id: leader.id, status: "accepted")

    visit "/feed"

    fill_in "comment[body]", with: "New comment"
    click_button "Create Comment"

    expect(page).to have_content("New comment")
  end

  it "allows the user to delete their comment", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    leader = User.create(username: "leader", email: "leader@example.com", password: "appdev", private: false)
    photo = create_photo(owner: leader)
    FollowRequest.create(sender_id: user.id, recipient_id: leader.id, status: "accepted")
    comment = Comment.create(body: "New comment", author_id: user.id, photo_id: photo.id)

    visit "/feed"

    within("li", text: comment.body) do
      click_on "Delete"
    end

    expect(page).not_to have_content("New comment")
  end

  it "allows the user to edit their comment", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    leader = User.create(username: "leader", email: "leader@example.com", password: "appdev", private: false)
    photo = create_photo(owner: leader)
    FollowRequest.create(sender_id: user.id, recipient_id: leader.id, status: "accepted")
    comment = Comment.create(body: "New comment", author_id: user.id, photo_id: photo.id)

    visit "/feed"

    within("li", text: comment.body) do
      click_on "Edit"
    end

    fill_in "comment[body]", with: "Edited comment"
    click_button "Update Comment"

    expect(page).to have_content("Edited comment")
  end

  it "shows an alert instead of saving a blank comment", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    leader = User.create(username: "leader", email: "leader@example.com", password: "appdev", private: false)
    create_photo(owner: leader)
    FollowRequest.create(sender_id: user.id, recipient_id: leader.id, status: "accepted")

    visit "/feed"

    click_button "Create Comment"

    expect(page).to have_content("Body can't be blank")
    expect(page).to have_current_path("/feed")
  end
end

def sign_in(user)
  visit "/users/sign_in"

  fill_in "Email", with: user.email
  fill_in "Password", with: user.password
  click_button "Sign in"
end

def create_photo(owner:, caption: "caption")
  photo = Photo.new(caption: caption, owner_id: owner.id)
  photo.image.attach(io: File.open(Rails.root.join("spec/support/test_image.jpeg")), filename: "test_image.jpeg", content_type: "image/jpeg")
  photo.save!
  photo
end

-->
