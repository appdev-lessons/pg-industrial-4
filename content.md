# Photogram Industrial: Building out the views

## Getting started

Let's continue building our Photogram Industrial project. Here's the target we're working towards:

[pg-industrial.matchthetarget.com](https://pg-industrial.matchthetarget.com/)

Navigate to `github.com/codespaces` (or reopen the previous lesson and use the "Load assignment" button) and reopen your `pg-industrial` project codespace to continue building on what you accomplished in the previous lessons.

At this point, you should have a fully navigable app with working authentication, a responsive three-column layout with sidebar navigation, mobile navigation, a photo upload modal, customized scaffold controllers, and a `UsersController` with stub view templates. What we _don't_ have yet are the real view templates: the photo card, the feed, the user profile page, follower lists, and customized Devise views. That's what we'll build in this lesson.

Make sure your sample data is loaded before continuing:

```
rake sample_data
```

Since each lesson builds on the work from the previous one, we want to branch off of our last branch rather than `main`. First, check out the branch from the previous lesson:

```
git checkout routes-layout-controllers
```

Now create a new branch from there:

```
git checkout -b profile-page-and-views
```

If you need a reference while you work, you can visit [my pull request](#my-pull-request){: target="_self" } below. Individual commits are also linked throughout the lesson for convenience.

## Custom image CSS

Before we start building views, note that `app/assets/stylesheets/custom-image.css` is already included in your project. It defines `.img-cover`, `.img-small`, `.img-medium`, and `.img-large` classes that we'll use throughout our view templates for avatars and profile images. You don't need to create this file; it's already there.

Now let's set up our branch and commit. This initial commit just establishes the branch:

```
git add -A
git commit -m "Started profile-page-and-views branch"
git push --set-upstream origin profile-page-and-views
```

That last command publishes the branch to GitHub. From now on, you can push with just `git push` after each commit.

[See my commit for this step.](https://github.com/bpurinton/pg-industrial/commit/)

### Open and submit your pull request

Now that you've pushed your branch to GitHub, it's time to open a **pull request** (PR). A pull request lets us review your code and leave line-by-line feedback.

<div class="alert alert-info">

[Here is a short video demonstration of the process.](https://share.descript.com/view/RLP4apAu5pp) You should also carefully read the notes below!
</div>

On GitHub, navigate to your `pg-industrial` repository. You should see a prompt to open a pull request for the `profile-page-and-views` branch that you just published, or you can go to the "Pull requests" tab and click "New pull request."

Make sure the **base** branch is `routes-layout-controllers` and the **compare** branch is `profile-page-and-views`. Also be sure to change the base _repository_ from `appdev-projects/pg-industrial` to _your_ fork. Give it a title, then click "Create pull request."

Your PR URL should look like:

```
github.com/[YOUR_GITHUB_USERNAME]/pg-industrial/pull/X
```

It should _not_ contain `appdev-projects` in the URL. If it does, you submitted a PR to _our_ repo instead of to _your own fork_.

<aside>
You don't need to merge your branches now, but when you're ready: [see the notes in our Git CLI lesson](/lessons/196-git-cli#merging-branches).
</aside>

Any additional commits you push to this branch will automatically show up on the pull request. As you work through the rest of this lesson, you can periodically check the PR diff on GitHub in the "Files changed" tab to get a comprehensive overview of everything that has changed so far.

Submit your pull request URL:

- `profile-page-and-views` compared to `routes-layout-controllers`:
- github.com
  - Great job!
- any
  - Not quite. Make sure the URL looks like: `github.com/[YOUR_GITHUB_USERNAME]/pg-industrial/pull/X`
{: .free_text #pr_url title="Pull request URL" points="1" answer="1" }

<div class="alert alert-info">
Are you still navigating manually through the file tree and clicking to open everything? That's going to become very painful, very quickly. One of the biggest things you can do to increase your productivity is navigating your codebase and its dozens of files without your mouse.

Stop now and experiment with [jumping to files](/lessons/194-helper-methods-part-3#partials-shine-along-with-jump-to-file) in the VSCode fuzzy search bar.
</div>

## List group layout partial

We'll be rendering collections of photos wrapped in `<li>` elements throughout the app. Rather than repeating the wrapping markup, let's create a layout partial that Rails can use with `render ... layout:`.

Create `app/views/layouts/_list_group.html.erb`:

```erb
<li class="list-group-item list-group-action-item">
  <%= yield %>
</li>
```
{: filename="app/views/layouts/_list_group.html.erb" }

When you pass `layout: "layouts/list_group"` to a `render partial: ... collection:` call, Rails wraps each rendered partial in this layout. The `<%= yield %>` is where the partial's content gets inserted. This keeps our view code DRY by defining the wrapping `<li>` once and reusing it everywhere.

## Follow/unfollow partial

Almost every page shows a Follow/Following/Requested button next to usernames. Let's build that reusable partial so it's ready when we need it.

This partial handles three states for the relationship between two users:

1. **Following**: the sender already follows the recipient (accepted follow request)
2. **Requested**: the sender has sent a pending follow request
3. **Follow**: no follow request exists yet

Create `app/views/follow_requests/_follow_unfollow.html.erb`. We'll walk through the logic in two parts.

First, the guard clause, lookup, and the two states for when a follow request already exists:

```erb{1-16}
<div>
  <% unless sender == recipient %>
    <% follow_request = sender.sent_follow_requests.find_by(recipient: recipient) %>

    <% if follow_request %>
      <% if follow_request.pending? %>
        <%= button_to follow_request, method: :delete, class: "btn btn-primary rounded-pill icon-link" do %>
          <i class="fa-solid fa-envelope"></i>
          Requested
        <% end %>
      <% elsif follow_request.accepted? %>
        <%= button_to follow_request, method: :delete, class: "btn btn-primary rounded-pill icon-link" do %>
          <i class="fa-solid fa-check"></i>
          Following
        <% end %>
      <% end %>
    <!-- ... -->
```
{: filename="app/views/follow_requests/_follow_unfollow.html.erb" }

- `unless sender == recipient`: you shouldn't see a follow button on your own profile!
- We look up whether the sender has an existing follow request for this recipient using `find_by`.
- If a request exists and is `pending?` (from our enum), we show a "Requested" button with an envelope icon. Clicking it sends a DELETE request to cancel the follow request.
- If a request exists and is `accepted?`, we show a "Following" button with a check icon. Clicking it sends a DELETE request to unfollow.

If no follow request exists, render the follow request form:

```erb{5-9}
    <!-- ... -->
          Following
        <% end %>
      <% end %>
    <% else %>
      <%= render "follow_requests/form", follow_request: recipient.received_follow_requests.build %>
    <% end %>
  <% end %>
</div>
```
{: filename="app/views/follow_requests/_follow_unfollow.html.erb" }

The `pending?` and `accepted?` methods come for free from our `enum :status` declaration on the FollowRequest model in an earlier lesson.

### Follow request form

Replace the scaffold-generated `app/views/follow_requests/_form.html.erb`:

```erb
<%= form_with(model: follow_request) do |form| %>
  <%= form.hidden_field :recipient_id %>

  <div>
    <%= form.button class: "btn btn-primary rounded-pill icon-link" do %>
      <% if follow_request.persisted? %>
        Following
      <% else %>
        <i class="fa-solid fa-plus"></i>
        Follow
      <% end %>
    <% end %>
  </div>
<% end %>
```
{: filename="app/views/follow_requests/_form.html.erb" }

We use `form.button ... do ... end` (block form) to include both the Font Awesome icon and the text inside the button. The `persisted?` check differentiates between an existing follow request and a new one being built.

Recall from the previous lesson that our `FollowRequestsController#create` action auto-accepts the request if the recipient's account is public. So clicking "Follow" on a public account will immediately change to "Following" on the next page load.

Commit:

```
git add -A
git commit -m "Added follow/unfollow partial and form"
git push
```

[See my commit for this step.](https://github.com/bpurinton/pg-industrial/commit/)

## Comment partial

Comments appear below every photo. Let's build the comment display and the form for adding new ones.

Each comment renders with the author's avatar, display name, username, time, comment body, and a dropdown for edit/delete. Replace the scaffold-generated `app/views/comments/_comment.html.erb`. We'll build it in two parts.

Start with the outer structure, avatar, author info, and comment body. Notice the `<li>` uses `dom_id(comment)` (e.g., `comment_42`) as its HTML id. This is important for Capybara tests that use `within("#comment_42")` to scope actions to a specific comment:

```erb{1-21}
<li id="<%= dom_id(comment) %>" class="list-group-item">
  <div class="p-3 pb-0">
    <div class="d-flex">
      <div class="flex-shrink-0">
        <%= image_tag comment.author.avatar_image, class: "rounded-circle img-small" %>
      </div>
      <div class="flex-grow-1 ms-3">
        <h6>
          <%= link_to user_path(comment.author.username), class: "text-decoration-none" do %>
            <%= comment.author.display_name %>
            <span class="fw-lighter text-body">
              @<%= comment.author.username %>
            </span>
          <% end %>
          &middot;
          <%= time_ago_in_words(comment.created_at) %>
        </h6>
        <p class="mb-0">
          <%= comment.body %>
        </p>

        <!-- ... -->
```
{: filename="app/views/comments/_comment.html.erb" }

This uses the Bootstrap flex media object pattern: a flex container with the avatar (`flex-shrink-0`) on the left and the content (`flex-grow-1`) on the right.

Next, add the dropdown menu at the bottom-right for edit and delete actions:

```erb{6-31}
        <!-- ... -->
        <p class="mb-0">
          <%= comment.body %>
        </p>

        <div class="d-flex justify-content-end">
          <div class="dropdown">
            <%= button_tag class: "btn btn-link text-decoration-none",
            data: {bs_toggle: "dropdown"},
            aria: { expanded: false } do %>
              <i class="fa-solid fa-ellipsis"></i>
            <% end %>
            <ul class="dropdown-menu">
              <li>
                <%= link_to edit_comment_path(comment), class: "dropdown-item" do %>
                  Edit
                <% end %>
              </li>
              <li>
                <%= button_to comment_path(comment), method: :delete, class: "dropdown-item" do %>
                  Delete
                <% end %>
              </li>
            </ul>
          </div>
        </div>

      </div>
    </div>
  </div>
</li>
```
{: filename="app/views/comments/_comment.html.erb" }

## Comment form

Replace the scaffold-generated `app/views/comments/_form.html.erb`:

```erb
<%= form_with(model: comment) do |form| %>
  <%= form.hidden_field :photo_id %>

  <div class="form-group">
    <%= form.label :body, class: "visually-hidden" %>
    <%= form.text_area :body, class: "form-control" %>
  </div>

  <div class="d-grid gap-2 mb-3">
    <%= form.submit class: "btn btn-primary" %>
  </div>
<% end %>
```
{: filename="app/views/comments/_form.html.erb" }

We hide the `photo_id` in a hidden field (it's auto-filled from the photo the comment belongs to) and hide the label with `visually-hidden` (still accessible to screen readers, but not visible). The form is clean: just a text area and a submit button.

## Like/unlike partial

The like button toggles between a solid heart (already liked) and an outline heart (not yet liked). Let's build both states.

Create `app/views/photos/_likes.html.erb`:

```erb
<div>
  <% like = current_user.likes.find_by(photo: photo) %>
  <% if like %>
    <%= button_to like, method: :delete, class: "btn btn-link icon-link text-decoration-none" do %>
      <i class="fa-solid fa-heart"></i>
      <%= pluralize(photo.likes_count, "like") %>
    <% end %>
  <% else %>
    <%= render "likes/form", like: photo.likes.build(fan: current_user) %>
  <% end %>
</div>
```
{: filename="app/views/photos/_likes.html.erb" }

The logic is straightforward:

1. Look up whether the current user already has a like for this photo.
2. If they do, show a **solid heart** (`fa-solid fa-heart`) with a `button_to` that sends a DELETE request to destroy the like (un-like).
3. If they don't, render the like form with an **outline heart** that creates a new like when clicked.

### Like form partial

Replace the scaffold-generated `app/views/likes/_form.html.erb`:

```erb
<%= form_with(model: like) do |form| %>
  <%= form.hidden_field :photo_id %>

  <%= form.button class: "btn btn-link icon-link text-decoration-none" do %>
    <i class="fa-regular fa-heart"></i>
    <%= pluralize(like.photo.likes_count, "like") %>
  <% end %>
<% end %>
```
{: filename="app/views/likes/_form.html.erb" }

The `photo_id` is passed as a hidden field so the `LikesController#create` action knows which photo to like. We use `form.button ... do ... end` (block form) to include both the Font Awesome icon and the count text inside the button.

### Like partial (for likes index page)

Replace the scaffold-generated `app/views/likes/_like.html.erb`:

```erb
<div id="<%= dom_id(like) %>">
  <div class="d-flex">
    <div class="flex-shrink-0">
      <%= image_tag like.fan.avatar_image, class: "rounded-circle img-small" %>
    </div>
    <div class="flex-grow-1 ms-3">
      <div class="d-flex justify-content-between">

        <div class="">
          <%= like.fan.display_name %>
          <div class="fw-lighter">
            @<%= like.fan.username %>
          </div>
        </div>

        <%= render "follow_requests/follow_unfollow", sender: current_user, recipient: like.fan %>
      </div>

      <p class="mb-0">
        <%= like.fan.bio %>
      </p>

    </div>
  </div>
</div>
```
{: filename="app/views/likes/_like.html.erb" }

This partial is used on the "liked by" page (`/photos/:id/likes`) to show each user who liked a photo. It uses the Bootstrap media object pattern: a flex container with the avatar on the left and the user's info on the right, including a follow/unfollow button.

Now let's commit all the partials we've created so far:

```
git add -A
git commit -m "Added comment, like, and supporting partials"
git push
```

[See my commit for this step.](https://github.com/bpurinton/pg-industrial/commit/)

## The photo card partial

This is the most important partial in the app. Every photo on the feed, discover, and profile pages uses this same card. Let's build it piece by piece.

Replace the scaffold-generated `app/views/photos/_photo.html.erb`. We'll build this file section by section.

### Pinned indicator

First, wrap the entire partial in a `dom_id` div and add the pinned indicator:

```erb{1-7}
<div id="<%= dom_id(photo) %>">
  <% if photo.pinned? && current_page?(user_path(photo.owner.username)) %>
    <div class="icon-link text-primary">
      <i class="fa-solid fa-thumbtack"></i>
      Pinned
    </div>
  <% end %>
  <!-- ... -->
```
{: filename="app/views/photos/_photo.html.erb" }

We only show the "Pinned" badge when we're on the owner's profile page. The `current_page?` helper checks whether the current URL matches the given path. This way, pinned photos get a visual indicator on profile pages but not on the feed or discover pages.

### Owner info and follow button

Next, add the photo owner's avatar, username link, and a follow/unfollow button:

```erb{5-12}
  <!-- ... -->
      Pinned
    </div>
  <% end %>
  <div class="h5 m-0 p-0 d-flex align-items-center justify-content-between mb-2">
    <div class="d-flex">
      <%= image_tag photo.owner.avatar_image, class: "rounded-circle me-2 img-cover img-small" %>

      <%= link_to photo.owner.username, user_path(photo.owner.username), class: "text-body link-underline-secondary link-underline-opacity-0 link-underline-opacity-100-hover" %>
    </div>
    <%= render "follow_requests/follow_unfollow", sender: current_user, recipient: photo.owner %>
  </div>
  <!-- ... -->
```
{: filename="app/views/photos/_photo.html.erb" }

We reuse the `follow_requests/follow_unfollow` partial we just created, passing `current_user` as the `sender` and `photo.owner` as the `recipient`.

### Image and likes count

Display the photo image and a link showing the likes count:

```erb{5-12}
  <!-- ... -->
    <%= render "follow_requests/follow_unfollow", sender: current_user, recipient: photo.owner %>
  </div>

  <div>
    <%= image_tag photo.image, class: "img-fluid w-100" %>
  </div>
  <div class="d-flex">
    <%= link_to photo.likes_count, photo_likes_path(photo), class: "link-primary text-decoration-none pe-1" %>
    <span>
      <%= "like".pluralize(photo.likes_count) %>
    </span>
  </div>
  <!-- ... -->
```
{: filename="app/views/photos/_photo.html.erb" }

The photo uses `img-fluid w-100` to fill the card width responsively. Below it, the likes count links to the photo's likes page (the nested route `/photos/:id/likes`), and the `pluralize` helper outputs "1 like" or "3 likes" as appropriate.

### Action buttons

Add the like button, comment link, and a [Bootstrap dropdown](https://getbootstrap.com/docs/5.3/components/dropdowns/) menu for edit/delete/pin:

```erb{5-38}
  <!-- ... -->
    </span>
  </div>

  <div class="d-flex justify-content-between">
    <div class="d-flex">
      <%= render "photos/likes", photo: photo %>
      <%= link_to photo_path(photo), class: "btn btn-link icon-link text-decoration-none" do %>
        <i class="fa-regular fa-comment"></i>
        <%= photo.comments_count %>
      <% end %>
    </div>
    <div class="d-flex">
      <div class="dropdown">
        <%= button_tag class: "btn btn-link text-decoration-none",
        data: { bs_toggle: "dropdown"}, aria: {expanded: false} do %>
          <i class="fa-solid fa-ellipsis"></i>
        <% end %>
        <ul class="dropdown-menu">
          <li>
            <%= link_to edit_photo_path(photo), class: "dropdown-item" do %>
              Edit
            <% end %>
          </li>
          <li>
            <%= button_to photo_path(photo), method: :delete, class: "dropdown-item" do %>
              Delete
            <% end %>
          </li>
          <li>
            <%= button_to photo_path(photo), method: :patch, class: "dropdown-item", params: { photo: { pinned: !photo.pinned } } do %>
              <%= photo.pinned? ? "Un-pin" : "Pin" %>
            <% end %>
          </li>
        </ul>
      </div>
    </div>
  </div>
  <!-- ... -->
```
{: filename="app/views/photos/_photo.html.erb" }

On the left side, we render the like button (a separate partial) and a comment icon linking to the photo's show page. The dropdown menu on the right has three options:

- **Edit**: links to the edit page
- **Delete**: sends a DELETE request via `button_to`
- **Pin/Un-pin**: sends a PATCH request that toggles the `pinned` boolean. The `params: { photo: { pinned: !photo.pinned } }` syntax flips the current value.

### Caption and comments

Finally, add the caption area with the owner's info and timestamp, followed by the comments list and comment form:

```erb{5-27}
  <!-- ... -->
    </div>
  </div>

  <p>
    <%= link_to user_path(photo.owner.username), class: "text-decoration-none" do %>
      <%= photo.owner.display_name %>
      <span class="fw-lighter text-body">
        @<%= photo.owner.username %>
      </span>
    <% end %>
    <span class="fw-lighter">
      &middot;
      <%= time_ago_in_words(photo.created_at) %>
    </span>
    <p>
      <%= photo.caption %>
    </p>
  </p>

  <ul class="list-group list-group-flush">
    <%= render photo.comments.default_order %>
    <li class="list-group-item mt-2">
      <%= render "comments/form", comment: photo.comments.build %>
    </li>
  </ul>
</div>
```
{: filename="app/views/photos/_photo.html.erb" }

The caption area shows the owner's display name, username, a relative timestamp (via `time_ago_in_words`), and the caption text. Below that, we render all comments in a [`list-group`](https://getbootstrap.com/docs/5.3/components/list-group/) using `render photo.comments.default_order` (which uses the `default_order` scope from an earlier lesson to show oldest-first), followed by a comment form for adding a new comment.

<aside markdown="1">
Notice that `render photo.comments.default_order` uses Rails' convention: when you pass an ActiveRecord collection to `render`, Rails automatically looks for a partial named after the model (`comments/_comment.html.erb`) and renders it once for each record, passing the local variable `comment`. This is equivalent to `render partial: "comments/comment", collection: photo.comments.default_order`.
</aside>

Commit:

```
git add -A
git commit -m "Built photo card partial with all dependencies"
git push
```

[See my commit for this step.](https://github.com/bpurinton/pg-industrial/commit/)

## Feed page

Let's put the photo card to use. Replace the feed stub with a real template that renders photo cards.

Replace `app/views/users/feed.html.erb`:

```erb
<% content_for :title, "Feed" %>

<h1>Feed</h1>
<hr>

<div id="photos">
  <ul id="feed" class="list-group list-group-flush">
    <%= render partial: "photos/photo", collection: @photos, layout: "layouts/list_group" %>
  </ul>
</div>
```
{: filename="app/views/users/feed.html.erb" }

The `render partial: ... collection: ... layout:` pattern renders the photo card once for each item in `@photos`, wrapping each in our `_list_group.html.erb` layout. The `@photos` variable comes from the `UsersController#feed` action, which returns all photos posted by people the user follows.

<div class="alert alert-success">

**CHECK**: Visit `/` or `/<username>/feed`. You should see real photo cards with images, like buttons, comment forms, and follow buttons! Try liking a photo and watch the heart fill in. Try adding a comment and see it appear below the photo. This is the moment the app comes alive.
</div>

Commit:

```
git add -A
git commit -m "Built feed page with photo cards"
git push
```

[See my commit for this step.](https://github.com/bpurinton/pg-industrial/commit/)

## Discover page

The discover page shows photos liked by people you follow. Replace the stub with a real template.

Replace `app/views/users/discover.html.erb`:

```erb
<% content_for :title, "Discover" %>

<h1>Discover</h1>
<hr>

<div id="photos">
  <ul class="list-group list-group-flush">
    <%= render partial: "photos/photo", collection: @photos, layout: "layouts/list_group" %>
  </ul>
</div>
```
{: filename="app/views/users/discover.html.erb" }

This is nearly identical to the feed page and reuses the same photo card partial. The difference is where `@photos` comes from: the `feed` association traverses User → Leaders → Own Photos, while `discover` traverses User → Leaders → Liked Photos. All that complex SQL is handled by the associations we defined in an earlier lesson.

<div class="alert alert-success">

**CHECK**: Visit `/<username>/discover`. You should see photos liked by people you follow, rendered as photo cards.
</div>

## User profile page

Visit `/<username>` in [the target](https://pg-industrial.matchthetarget.com). There's a banner image, circular avatar, display name, follower/following counts, and tabbed posts/likes sections. Let's build all of that.

The route `get ":username" => "users#show", as: :user` maps URLs like `/alice` to the `UsersController#show` action, which uses `find_by!(username: params[:username])` to look up the user.

Replace `app/views/users/show.html.erb`. We'll build this file section by section.

### Title and profile banner

Start with the page title and the profile banner area:

```erb{1-6}
<% content_for :title, "@#{@user.username}'s profile" %>

<div class="p-5 mb-4 bg-body-tertiary img-cover" style="<%= "background-image: url('#{url_for(@user.profile_banner)}')" if @user.profile_banner.attached? %>">
  <div class="container-fluid py-5">
  </div>
</div>
<!-- ... -->
```
{: filename="app/views/users/show.html.erb" }

The profile banner uses a `div` with a CSS `background-image` property, but only if the user has a banner attached. We use `url_for()` to generate the Active Storage URL. If no banner is attached, the div shows the `bg-body-tertiary` background color as a placeholder.

### Avatar

Next, add the avatar image that overlaps the banner:

```erb{6}
<!-- ... -->
  <div class="container-fluid py-5">
  </div>
</div>

<%= image_tag @user.avatar_image, class: "me-3 rounded-circle img-cover img-medium border border-light border-3", style: "margin-top: -6rem;" %>

<!-- ... -->
```
{: filename="app/views/users/show.html.erb" }

The avatar uses `image_tag` with our custom `img-cover img-medium` classes for consistent sizing. The `style: "margin-top: -6rem;"` pulls the avatar up to overlap with the banner area. We don't need to check `attached?` for the avatar because our `before_create :set_default_avatar` callback ensures every user always has one.

<div class="alert alert-success">

**CHECK**: Visit `/<username>`. You should see a banner area and a circular avatar overlapping it.
</div>

### Display name, private badge, and follow button

Add the display name heading with a private [badge](https://getbootstrap.com/docs/5.3/components/badge/) and follow/unfollow button on the right using [flexbox utilities](https://getbootstrap.com/docs/5.3/utilities/flex/):

```erb{4-19}
<!-- ... -->
<%= image_tag @user.avatar_image, class: "me-3 rounded-circle img-cover img-medium border border-light border-3", style: "margin-top: -6rem;" %>

<h1 class="d-flex justify-content-between">
  <%= @user.display_name || @user.username %>
  <div class="d-flex justify-content-between">
    <% if @user.private %>
      <span class="d-flex align-items-center fs-6 badge text-bg-light border-dark fw-normal">
        Private
        <i class="fa-solid fa-lock"></i>
      </span>
    <% end %>

    <%= render "follow_requests/follow_unfollow", sender: current_user, recipient: @user %>
  </div>
</h1>
<h4>
  @<%= @user.username %>
</h4>
<!-- ... -->
```
{: filename="app/views/users/show.html.erb" }

We display `display_name` if present, otherwise fall back to `username`.

### Profile stats

Add the followers, following, pending, and posts counts, each linking to their respective pages:

```erb{5-41}
<!-- ... -->
  @<%= @user.username %>
</h4>

<span>
  <%= link_to followers_path(@user.username), class: "text-decoration-none" do %>
    <span class="text-primary fw-bold"><%= @user.followers.count %></span>
    <span class="text-body">
      followers
    </span>
  <% end %>
</span>

<span>
  <%= link_to follows_path(@user.username), class: "text-decoration-none" do %>
    <span class="text-primary fw-bold"><%= @user.leaders.count %></span>
    <span class="text-body">
      following
    </span>
  <% end %>
</span>

<% if current_user == @user && @user.private? %>
  <span>
    <%= link_to pending_path(@user.username), class: "text-decoration-none" do %>
      <span class="text-primary fw-bold">
        <%= @user.pending_received_follow_requests.count %>
      </span>
      <span class="text-body">
        pending
      </span>
    <% end %>
  </span>
<% end %>

<span>
  <span class="text-primary fw-bold">
  <%= @user.photos_count %>
  </span>
  posts
</span>
<!-- ... -->
```
{: filename="app/views/users/show.html.erb" }

A few details to notice:

- We use `link_to ... do ... end` (block form) to wrap both the count number and the label text inside a single link.
- The "pending" count only shows when `current_user == @user && @user.private?`, since you can only see your own pending requests, and only if your account is private.
- We use `@user.photos_count` (the counter cache column) instead of `@user.own_photos.count` to avoid an extra database query.

<div class="alert alert-success">

**CHECK**: Visit `/<username>`. You should see the display name, username, and stats (followers, following, posts counts) with clickable links.
</div>

### Bio and website

Add the user's bio and website link:

```erb{5-11}
<!-- ... -->
  posts
</span>

<p class="mt-2">
  <%= @user.bio %>
</p>

<%= link_to @user.website, @user.website, target: "_blank" %>

<hr>
<!-- ... -->
```
{: filename="app/views/users/show.html.erb" }

### Tabbed interface

Finally, add [Bootstrap 5's JavaScript-powered tabs](https://getbootstrap.com/docs/5.3/components/navs-tabs/#javascript-behavior) for Posts and Likes:

```erb{4-30}
<!-- ... -->
<hr>

<ul class="nav nav-underline mb-3" id="myTab" role="tablist">
  <li class="nav-item" role="presentation">
    <button class="nav-link active" id="posts-tab" data-bs-toggle="tab" data-bs-target="#posts-tab-pane" type="button" role="tab" aria-controls="posts-tab-pane" aria-selected="true">Posts <span class="badge text-bg-secondary"><%= @user.photos_count %></span></button>
  </li>
  <li class="nav-item" role="presentation">
    <button class="nav-link" id="likes-tab" data-bs-toggle="tab" data-bs-target="#likes-tab-pane" type="button" role="tab" aria-controls="likes-tab-pane" aria-selected="false">Likes <span class="badge text-bg-secondary"><%= @user.likes_count %></span></button>
  </li>
</ul>
<div class="tab-content" id="myTabContent">
  <div class="tab-pane fade show active" id="posts-tab-pane" role="tabpanel" aria-labelledby="posts-tab" tabindex="0">
    <% if @user.photos_count.positive? %>
      <ul class="list-group list-group-flush">
        <%= render partial: "photos/photo", collection: @user.own_photos.pinned, layout: "layouts/list_group" %>
        <%= render partial: "photos/photo", collection: @user.own_photos.unpinned, layout: "layouts/list_group" %>
      </ul>
    <% else %>
      <div class="fs-4 text-muted">
        User hasn't posted anything yet.
      </div>
    <% end %>
  </div>
  <div class="tab-pane fade" id="likes-tab-pane" role="tabpanel" aria-labelledby="likes-tab" tabindex="0">
    <ul class="list-group list-group-flush">
      <%= render partial: "photos/photo", collection: @user.liked_photos, layout: "layouts/list_group" %>
    </ul>
  </div>
</div>
```
{: filename="app/views/users/show.html.erb" }

The `<button>` elements have `data-bs-toggle="tab"` and `data-bs-target` attributes that tell Bootstrap which content pane to show when clicked. No custom JavaScript needed; Bootstrap handles it.

In the **Posts** tab pane, we render pinned photos first, then unpinned photos. This uses the `pinned` and `unpinned` scopes we defined on the Photo model in an earlier lesson. Each photo is wrapped in a list group item using our `layouts/list_group` layout partial.

In the **Likes** tab pane, we render all of the user's liked photos using the `liked_photos` association.

<aside markdown="1">
The `render partial: ... collection: ... layout:` pattern is a powerful Rails feature. It renders the partial once for each item in the collection, wrapping each in the specified layout. The local variable name inside the partial is automatically derived from the partial name, so `photos/_photo.html.erb` receives `photo` as a local.
</aside>

<div class="alert alert-success">

**CHECK**: Visit `/<username>`. You should see the complete profile page with banner, avatar, stats, bio, and a tabbed interface. Click "Posts" to see the user's photos (pinned first). Click "Likes" to see photos they've liked.
</div>

Commit:

```
git add -A
git commit -m "Built user profile page with tabbed interface"
git push
```

[See my commit for this step.](https://github.com/bpurinton/pg-industrial/commit/)

## User list item partial

The followers, following, and search pages all show lists of users. Let's build a reusable partial before we tackle those pages.

Create `app/views/users/_list_item.html.erb`:

```erb
<li class="list-group-item list-group-action">
  <div class="d-flex">
    <div class="flex-shrink-0">
      <%= image_tag user.avatar_image, class: "rounded-circle img-small img-cover" %>
    </div>
    <div class="flex-grow-1 ms-3">
      <div class="d-flex justify-content-between">

        <div class="">
          <%= link_to user_path(user.username), class: "text-decoration-none" do %>
            <%= user.display_name %>
            <div class="fw-lighter text-body">
              @<%= user.username %>
            </div>
          <% end %>
        </div>

        <%= render "follow_requests/follow_unfollow", sender: current_user, recipient: user %>
      </div>

      <p class="mb-0">
        <%= user.bio %>
      </p>

    </div>
  </div>
</li>
```
{: filename="app/views/users/_list_item.html.erb" }

This uses the [Bootstrap flex](https://getbootstrap.com/docs/5.3/utilities/flex/) media object pattern: avatar on the left, user info and follow button on the right, with the bio below. The display name links to the user's profile page using our vanity URL route.

## Followers page

Replace `app/views/users/followers.html.erb`:

```erb
<% content_for :title, "Followers" %>

<h1>Followers</h1>
<small><%= @followers.count %> followers</small>

<div class="fs-5 border-bottom mb-2">
  <%= link_to :back, class: "text-decoration-none icon-link" do %>
    <i class="fa-solid fa-arrow-left"></i>
    Back
  <% end %>
</div>

<div>
  <ul class="list-group list-group-flush">
    <% @followers.each do |user| %>
      <%= render "users/list_item", user: user %>
    <% end %>

    <% if @followers.empty? %>
      <div class="fs-4 text-muted">
        User doesn't have any followers yet.
      </div>
    <% end %>
  </ul>
</div>
```
{: filename="app/views/users/followers.html.erb" }

The `link_to :back` generates a link to the previous page using the browser's referrer, a convenient Rails helper. Each follower is rendered using our `_list_item` partial, and we show a friendly message if the user has no followers.

<div class="alert alert-success">

**CHECK**: Visit `/<username>/followers`. You should see a list of followers with avatars, usernames, and follow/unfollow buttons.
</div>

## Following page

Replace `app/views/users/follows.html.erb`:

```erb
<% content_for :title, "Following" %>

<h1>Following</h1>
<small><%= @follows.count %> following</small>

<div class="fs-5 border-bottom mb-2">
  <%= link_to :back, class: "text-decoration-none icon-link" do %>
    <i class="fa-solid fa-arrow-left"></i>
    Back
  <% end %>
</div>

<div>
  <ul class="list-group list-group-flush">
    <% @follows.each do |user| %>
      <%= render "users/list_item", user: user %>
    <% end %>

    <% if @follows.empty? %>
      <div class="fs-4 text-muted">
        User hasn't followed anyone yet.
      </div>
    <% end %>
  </ul>
</div>
```
{: filename="app/views/users/follows.html.erb" }

This follows the same pattern as the followers page but uses `@follows` (the user's leaders).

<div class="alert alert-success">

**CHECK**: Visit `/<username>/follows`. You should see a list of people the user follows.
</div>

## Pending page

Unlike followers/following, pending requests need Accept and Reject buttons. This page is more complex because it needs to iterate over follow requests (not users) so we can build forms to update each request's status.

Replace `app/views/users/pending.html.erb`. We'll build it in three parts.

The header follows the same structure as the followers and following pages:

```erb{1-11}
<% content_for :title, "Pending" %>

<h1>Pending</h1>
<small><%= @pending.count %> pending</small>

<div class="fs-5 border-bottom mb-2">
  <%= link_to :back, class: "text-decoration-none icon-link" do %>
    <i class="fa-solid fa-arrow-left"></i>
    Back
  <% end %>
</div>
<!-- ... -->
```
{: filename="app/views/users/pending.html.erb" }

Next, the loop iterates over follow requests (not users) so we can build forms for each one. The user info layout mirrors `_list_item`, but instead of a Follow/Unfollow button, we show Accept and Reject forms:

```erb{5-50}
<!-- ... -->
  <% end %>
</div>

<div>
  <ul class="list-group list-group-flush">
    <% @pending.each do |follow_request| %>
      <% user = follow_request.sender %>
      <li class="list-group-item list-group-action">
        <div class="d-flex">
          <div class="flex-shrink-0">
            <%= image_tag user.avatar_image, class: "rounded-circle img-small img-cover" %>
          </div>
          <div class="flex-grow-1 ms-3">
            <div class="d-flex justify-content-between">
              <div class="">
                <%= link_to user_path(user.username), class: "text-decoration-none" do %>
                  <%= user.display_name %>
                  <div class="fw-lighter text-body">
                    @<%= user.username %>
                  </div>
                <% end %>
              </div>

              <div class="d-flex gap-3">
                <%= form_with(model: follow_request) do |form| %>
                  <%= form.hidden_field :recipient_id %>
                  <%= form.hidden_field :status, value: "accepted" %>

                  <div>
                    <%= form.button class: "btn btn-primary rounded-pill icon-link" do %>
                        <i class="fa-solid fa-check"></i>
                        Accept
                    <% end %>
                  </div>
                <% end %>

                <%= form_with(model: follow_request) do |form| %>
                  <%= form.hidden_field :recipient_id %>
                  <%= form.hidden_field :status, value: "rejected" %>

                  <div>
                    <%= form.button class: "btn btn-primary rounded-pill icon-link" do %>
                      <i class="fa-solid fa-times"></i>
                      Reject
                    <% end %>
                  </div>
                <% end %>
              </div>
            </div>
            <!-- ... -->
```
{: filename="app/views/users/pending.html.erb" }

The key thing here is the two `form_with` calls for each follow request. Both forms submit a PATCH request to `FollowRequestsController#update`, but they send different `status` values:

- The **Accept** form sends `status: "accepted"`, which triggers the `FollowRequest` to be accepted.
- The **Reject** form sends `status: "rejected"`, which rejects it.

Each form includes `hidden_field :recipient_id` and `hidden_field :status` to pass the necessary data. The `FollowRequestsController#update` action (set up in the previous lesson) handles updating the status accordingly.

Finally, close the loop with the user's bio and an empty state message:

```erb{5-20}
            <!-- ... -->
              </div>
            </div>

            <p class="mb-0">
              <%= user.bio %>
            </p>

          </div>
        </div>
      </li>
    <% end %>
    <% if @pending.empty? %>
      <div class="fs-4 text-muted">
        User doesn't have any pending follows.
      </div>
    <% end %>
  </ul>
</div>
```
{: filename="app/views/users/pending.html.erb" }

<aside markdown="1">
We can't reuse the `_list_item` partial here because we need the Accept/Reject buttons instead of the Follow/Unfollow button. When a partial doesn't quite fit, it's fine to inline the markup. Don't force a partial to do something it wasn't designed for.
</aside>

<div class="alert alert-success">

**CHECK**: Visit `/<username>/pending`. You should see pending follow requests with Accept and Reject buttons. Try accepting or rejecting one.
</div>

Commit:

```
git add -A
git commit -m "Added user list item partial, followers, following, and pending pages"
git push
```

[See my commit for this step.](https://github.com/bpurinton/pg-industrial/commit/)

## User search results

The users index page displays search results. The `UsersController#index` action uses Ransack to search by username. The view is simple since it reuses our `_list_item` partial.

Replace `app/views/users/index.html.erb`:

```erb
<div>
  <ul class="list-group list-group-flush">
    <% @users.each do |user| %>
      <%= render "users/list_item", user: user %>
    <% end %>
  </ul>
</div>
```
{: filename="app/views/users/index.html.erb" }

This renders a list group of users. The Ransack search form in the sidebar (from the application layout in the previous lesson) submits to this page, and the `@users` variable contains the results.

<div class="alert alert-success">

**CHECK**: Use the search bar in the right sidebar (visible on wide screens) and search for a username. You should see matching users with their avatars and follow buttons.
</div>

## Photo show and edit pages

Let's replace the scaffold-generated photo show and edit pages with cleaner versions.

Replace `app/views/photos/show.html.erb`:

```erb
<div class="fs-5 border-bottom mb-2">
  <%= link_to :back, class: "text-decoration-none icon-link" do %>
    <i class="fa-solid fa-arrow-left"></i>
    Back
  <% end %>
</div>

<%= render @photo %>
```
{: filename="app/views/photos/show.html.erb" }

This is beautifully simple. The `render @photo` line uses Rails' convention to automatically look for `photos/_photo.html.erb` and render it with `photo: @photo`. All the complexity lives in the partial.

Replace `app/views/photos/edit.html.erb`:

```erb
<% content_for :title, "Editing photo" %>

<h1>Editing photo</h1>

<%= render "form", photo: @photo %>

<br>

<div>
  <%= link_to "Back", :back %>
</div>
```
{: filename="app/views/photos/edit.html.erb" }

<div class="alert alert-success">

**CHECK**: Click on a photo from the feed or profile page → see the photo detail page with a Back link. Click Edit → see the edit form with the current image and caption.
</div>

## Photo likes page

When a user clicks on the likes count on a photo, they see a list of all users who liked that photo. This uses the nested route `/photos/:photo_id/likes`, which routes to `LikesController#index`.

Replace the scaffold-generated `app/views/likes/index.html.erb`:

```erb
<% content_for :title, "Liked by" %>

<h1>Liked by</h1>
<small><%= pluralize(@photo.likes_count, "like") %></small>
<hr>
<%= link_to "back", @photo %>
<div>
  <ul class="list-group list-group-flush mb-5">
    <% @likes.each do |like| %>
      <li class="list-group-item list-group-action">
        <%= render like %>
      </li>
    <% end %>
  </ul>
</div>
```
{: filename="app/views/likes/index.html.erb" }

Each like is rendered using the `likes/_like.html.erb` partial we created earlier, which shows the fan's avatar, display name, username, bio, and a follow/unfollow button.

<div class="alert alert-success">

**CHECK**: Click the likes count on any photo → see a list of users who liked it, each with a follow/unfollow button.
</div>

Commit:

```
git add -A
git commit -m "Added search results, photo show/edit, and likes pages"
git push
```

[See my commit for this step.](https://github.com/bpurinton/pg-industrial/commit/)

## Customizing Devise views

Up until now, we've been using Devise's built-in views for sign in, sign up, and profile editing. These work but look plain. To customize them, we need to generate the Devise view files into our project so we can edit them.

Run the Devise view generator:

```
rails generate devise:views users
```

<aside markdown="1">
We pass `users` because that's our Devise scope. This places the generated views in `app/views/users/` (e.g., `app/views/users/sessions/new.html.erb`, `app/views/users/registrations/new.html.erb`). If you just ran `rails generate devise:views` without specifying the scope, the views would go into `app/views/devise/`, which works but doesn't match our convention.
</aside>

This generates several files. The three we care about are:

- `app/views/users/sessions/new.html.erb`: the sign in form
- `app/views/users/registrations/new.html.erb`: the sign up form
- `app/views/users/registrations/edit.html.erb`: the settings/profile edit form

Before we customize these views, we need to tell Devise about our custom User fields. Otherwise, Devise will reject any extra fields we add to the sign up and edit forms.

### Permitted parameters

Open `app/controllers/application_controller.rb` and add a `before_action` for Devise's permitted parameters, along with a `protected` method:

```ruby{2,7,9-12}
class ApplicationController < ActionController::Base
  before_action :configure_permitted_parameters, if: :devise_controller?
  before_action :authenticate_user!
  before_action :set_user_search, if: -> { current_user.present? }

  # ...

  protected

  def configure_permitted_parameters
    devise_parameter_sanitizer.permit(:sign_up, keys: [:display_name, :username])
    devise_parameter_sanitizer.permit(:account_update, keys: [:avatar_image, :bio, :display_name, :username, :private, :profile_banner, :remove_profile_banner, :website])
  end
end
```
{: filename="app/controllers/application_controller.rb" }

The `if: :devise_controller?` condition means this `before_action` only runs when the request is being handled by one of Devise's built-in controllers (sign up, sign in, edit profile, etc.). It doesn't run for our custom controllers.

By default, Devise only permits `email`, `password`, and `password_confirmation`. Since we added custom columns to our User model, we need to explicitly tell Devise to allow them through. The `:sign_up` sanitizer controls which fields are accepted during registration, and the `:account_update` sanitizer controls which fields are accepted when editing a profile.

Notice that `:sign_up` only permits `:display_name` and `:username`, since we don't want users uploading avatars or setting bios during registration. Those are for the account update form. Also note `:remove_profile_banner` in the account update list. This is the virtual attribute we set up on the User model in an earlier lesson for removing the banner image via a checkbox.

### Sign in view

The default Devise sign in page has a "Log in" heading and button. Our tests expect "Sign in" instead. Open the generated file and replace it:

```erb
<h2>Sign in</h2>

<%= form_for(resource, as: resource_name, url: session_path(resource_name)) do |f| %>
  <%= render "users/shared/error_messages", resource: resource %>

  <div class="form-group">
    <%= f.label :email %><br />
    <%= f.email_field :email, autofocus: true, autocomplete: "email", class: "form-control" %>
  </div>

  <div class="form-group">
    <%= f.label :password %><br />
    <%= f.password_field :password, autocomplete: "current-password", class: "form-control" %>
  </div>

  <% if devise_mapping.rememberable? %>
    <div class="form-check mb-3">
      <%= f.label :remember_me, class: "form-check-label" do %>
        <%= f.check_box :remember_me, class: "form-check-input" %> Remember me
      <% end %>
    </div>
  <% end %>

  <div class="mt-2 actions d-grid gap-2">
    <%= f.submit "Sign in", class: "btn btn-primary" %>
  </div>
<% end %>

<%= render "users/shared/links" %>
```
{: filename="app/views/users/sessions/new.html.erb" }

The main changes from the default: we changed "Log in" to "Sign in" in both the heading and submit button, added `class: "form-control"` to inputs for [Bootstrap form styling](https://getbootstrap.com/docs/5.3/forms/form-control/), and added `class: "btn btn-primary"` to the submit button.

<div class="alert alert-success">

**CHECK**: Sign out and visit `/users/sign_in`. You should see a styled sign-in form with Bootstrap form controls.
</div>

### Sign up view

The sign up form needs two additional fields that Devise doesn't include by default: `display_name` and `username`. We already permitted these parameters in `ApplicationController` via `configure_permitted_parameters`.

```erb
<h2>Sign up</h2>

<%= form_for(resource, as: resource_name, url: registration_path(resource_name)) do |f| %>
  <%= render "users/shared/error_messages", resource: resource %>

  <div class="form-group">
    <%= f.label :email %><br />
    <%= f.email_field :email, autofocus: true, autocomplete: "email", class: "form-control" %>
  </div>

  <div class="form-group">
    <%= f.label :display_name %><br />
    <%= f.text_field :display_name, class: "form-control" %>
  </div>

  <div class="form-group">
    <%= f.label :username %><br />
    <%= f.text_field :username, class: "form-control" %>
  </div>

  <div class="form-group">
    <%= f.label :password %>
    <% if @minimum_password_length %>
    <em>(<%= @minimum_password_length %> characters minimum)</em>
    <% end %><br />
    <%= f.password_field :password, autocomplete: "new-password", class: "form-control" %>
  </div>

  <div class="form-group">
    <%= f.label :password_confirmation %><br />
    <%= f.password_field :password_confirmation, autocomplete: "new-password", class: "form-control" %>
  </div>

  <div class="mt-2 actions d-grid gap-2">
    <%= f.submit "Sign up", class: "btn btn-primary" %>
  </div>
<% end %>

<%= render "users/shared/links" %>
```
{: filename="app/views/users/registrations/new.html.erb" }

The key additions are the `display_name` and `username` fields between the email and password fields. These are already permitted through the `configure_permitted_parameters` method we set up in `ApplicationController`, so they'll be saved when the form is submitted.

<div class="alert alert-success">

**CHECK**: Visit `/users/sign_up`. You should see a sign-up form with Username and Display name fields alongside the standard Email and Password fields.
</div>

### Settings / profile edit view

The edit profile form is the most complex Devise view. It needs fields for changing the password, updating profile information (username, display name, bio, website, private toggle), and uploading images (avatar, profile banner). It also includes [Bootstrap validation feedback](https://getbootstrap.com/docs/5.3/forms/validation/) that shows green/red borders on fields after a failed submission.

Replace the generated `app/views/users/registrations/edit.html.erb`. We'll build this form section by section.

Start with the heading, a flag to track whether validation has occurred, and the form tag:

```erb{1-9}
<h2>Edit <%= resource_name.to_s.humanize %></h2>

<div class="card-body">
  <% was_validated = resource.errors.any? %>

  <% form_html_options = { method: :put, novalidate: true, class: "mb-3" } %>

  <%= form_for(resource, as: resource_name, url: registration_path(resource_name), html: form_html_options) do |f| %>
    <!-- ... -->
```
{: filename="app/views/users/registrations/edit.html.erb" }

<aside markdown="1">
The `novalidate: true` disables the browser's built-in HTML5 validation. We do this because we want to use our own server-side validation with Bootstrap's styling instead of the browser's default (and inconsistent) validation popups.
</aside>

The `was_validated` flag will be used by each field group to decide whether to show green/red borders.

Each field in this form follows the same validation pattern. Here's how it looks for the `current_password` field:

```erb{4-34}
    <!-- ... -->
  <%= form_for(resource, as: resource_name, url: registration_path(resource_name), html: form_html_options) do |f| %>

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

      <%= f.password_field :current_password, class: current_password_class, autocomplete: "off" %>

      <% if current_password_was_invalid %>
        <% resource.errors.full_messages_for(:current_password).each do |message| %>
          <div class="invalid-feedback d-flex">
            <%= message %>
          </div>
        <% end %>
      <% end %>

      <small class="form-text text-muted">
        We need your current password to confirm any changes.
      </small>
    </div>

    <hr class="mt-4">
    <!-- ... -->
```
{: filename="app/views/users/registrations/edit.html.erb" }

This pattern repeats for every field group:

1. Check if this specific field had an error: `resource.errors.include?(:field_name)`.
2. Start with a base CSS class: `"form-control"`.
3. If the form was submitted and validation failed (`was_validated`), add either `is-invalid` or `is-valid` to the class string.
4. Render the field with the computed class.
5. If the field was invalid, display the error messages in a `div.invalid-feedback`.

This gives users immediate visual feedback: green borders on valid fields, red borders and error messages on invalid ones. See the [Bootstrap validation docs](https://getbootstrap.com/docs/5.3/forms/validation/) for more on the `is-valid`, `is-invalid`, and `invalid-feedback` classes.

Use this same pattern for the **email**, **password**, **password\_confirmation**, **username**, **display\_name**, **bio**, and **website** fields, adjusting the field names and input types accordingly. Separate the password section from the profile info section with `<hr class="mt-4">` dividers. Current password is required by Devise for any changes. This is a security feature to prevent unauthorized edits from hijacked sessions.

The avatar field is unique because it uses `file_field` with a preview of the current avatar above it:

```erb{10-14}
    <!-- ... -->
    <hr class="mt-4">

    <div class="form-group">
      <% avatar_image_was_invalid = resource.errors.include?(:avatar_image) %>

      <!-- ... validation class logic same as above ... -->

      <%= f.label :avatar_image %>

      <div>
        <%= image_tag current_user.avatar_image, class: "img-cover img-large rounded-circle mb-2" %>

        <%= f.file_field :avatar_image, class: avatar_image_class, accept: "image/*" %>
      </div>

      <!-- ... error messages same as above ... -->
    </div>
    <!-- ... -->
```
{: filename="app/views/users/registrations/edit.html.erb" }

The `accept: "image/*"` attribute restricts the file picker to image files only.

The profile banner field also uses `file_field`, but adds a "Remove Profile Banner" checkbox:

```erb{10-23}
    <!-- ... -->
    </div>

    <div class="form-group">
      <% profile_banner_was_invalid = resource.errors.include?(:profile_banner) %>

      <!-- ... validation class logic same as above ... -->

      <%= f.label :profile_banner %>

      <div>
        <% if current_user.profile_banner.attached? %>
          <%= image_tag current_user.profile_banner, class: "img-cover w-100 mb-2" %>
        <% end %>

        <%= f.file_field :profile_banner, class: profile_banner_class, accept: "image/*" %>
      </div>

      <% if resource.profile_banner.attached? %>
        <div>
          <%= f.check_box :remove_profile_banner %>
          <%= f.label :remove_profile_banner, "Remove Profile Banner" %>
        </div>
      <% end %>

      <!-- ... error messages same as above ... -->
    </div>
    <!-- ... -->
```
{: filename="app/views/users/registrations/edit.html.erb" }

The "Remove Profile Banner" checkbox uses the `remove_profile_banner` virtual attribute we defined on the User model in an earlier lesson. When checked and the form is submitted, the `after_save :purge_profile_banner` callback removes the banner from Cloudinary.

The private field uses a checkbox instead of a text input:

```erb{8-11}
    <!-- ... -->
    </div>

    <div class="form-group">
      <% private_was_invalid = resource.errors.include?(:private) %>

      <!-- ... validation class logic (using "custom-control-input" as base class) ... -->

      <div class="custom-control custom-checkbox">
        <%= f.check_box :private, class: "#{private_class}" %>
        <%= f.label :private, class: "custom-control-label" %>
      </div>

      <!-- ... error messages same as above ... -->
    </div>
    <!-- ... -->
```
{: filename="app/views/users/registrations/edit.html.erb" }

Finally, close the form with an Update button and a Back link:

```erb{4-12}
    <!-- ... -->
    </div>

    <div class="d-grid">
      <%= f.submit "Update", class: "btn btn-outline-primary" %>
    </div>
  <% end %>

    <div class="d-grid">
      <%= link_to "Back", :back, class: "btn btn-outline-secondary" %>
    </div>
</div>
```
{: filename="app/views/users/registrations/edit.html.erb" }

<div class="alert alert-success">

**CHECK**: Visit `/users/edit` (or click "Settings" in the sidebar). You should see the full settings form with fields for current password, email, password change, username, display name, bio, website, avatar upload, banner upload, and private toggle. Try uploading a new avatar or changing your bio.
</div>

Commit:

```
git add -A
git commit -m "Customized Devise views: sign in, sign up, and settings"
git push
```

[See my commit for this step.](https://github.com/bpurinton/pg-industrial/commit/)

## Finish it off

We've built out the core views, but there may be some remaining details to polish before all the `grade` tests pass. Here are some hints:

- Make sure the navbar in your application layout (from the previous lesson) has links for "Feed", "Discover", "Go to profile", "Settings" (`/users/edit`), and "Sign out". The tests check for these.
- Make sure the "Add photo" button in the sidebar opens the new photo form (via the Bootstrap modal from the previous lesson).
- The photo form (`app/views/photos/_form.html.erb`) should use `form.file_field :image` for Active Storage uploads, not `form.text_field :image`.
- Run `grade` often and read the failing test names carefully. They tell you exactly what's expected.

If you get stuck, you can reference the target app at [pg-industrial.matchthetarget.com](https://pg-industrial.matchthetarget.com/) and look at [the solution pull request](https://github.com/appdev-projects/photogram-industrial/pull/4/files) for the complete code.

<div class="alert alert-info">

You do not need to get every visual detail of your app to match the target to get the `grade` tests to pass. The tests are more concerned with functionality: can you visit the right pages, see the right content, click the right buttons. Focus on making tests pass rather than pixel-perfect design. In the next project, _Photogram Industrial Authorization_, the starting point code will be a complete solution to the current target.
</div>

Now would be a good time for a final commit and push:

```
git add -A
git commit -m "Completed profile page, views, and Devise customization"
git push
```

[See my commit for this step.](https://github.com/bpurinton/pg-industrial/commit/)

## My pull request

You can visit the full diff of changes on my [pull request](https://github.com/bpurinton/pg-industrial/pull/4) in the [Files changed tab](https://github.com/bpurinton/pg-industrial/pull/4/files) if you need to compare your work to a working solution.

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

  it "has a bootstrap navbar", points: 1 do
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

    expect(page).to have_current_path("/#{user.username}/feed")
  end

  it "has a link, 'Discover', that navigates to the 'Discover' page", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    visit "/"

    click_on "Discover"

    expect(page).to have_current_path("/#{user.username}/discover")
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

  it "displays the user's username", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    visit "/#{user.username}"

    expect(page).to have_content("@#{user.username}")
  end

  it "has a link to followers page", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    visit "/#{user.username}"

    expect(page).to have_link(nil, href: "/#{user.username}/followers")
  end

  it "has a link to following page", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    visit "/#{user.username}"

    expect(page).to have_link(nil, href: "/#{user.username}/follows")
  end

  it "displays followers count", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    visit "/#{user.username}"

    expect(page).to have_content("0")
    expect(page).to have_content("followers")
  end

  it "displays following count", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    visit "/#{user.username}"

    expect(page).to have_content("0")
    expect(page).to have_content("following")
  end

  it "has a follow button for other users", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    other = User.create(username: "bob", email: "bob@example.com", password: "appdev")
    sign_in(user)

    visit "/#{other.username}"

    expect(page).to have_button("Follow")
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

  it "allows a user to accept a follow request", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    other_user = User.create(username: "other_user", email: "other_user@example.com", password: "appdev")
    FollowRequest.create(sender_id: other_user.id, recipient_id: user.id, status: "pending")

    visit "/#{user.username}/pending"

    click_on "Accept"

    expect(page).not_to have_content(other_user.username)
  end

  it "allows a user to reject a follow request", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    other_user = User.create(username: "other_user", email: "other_user@example.com", password: "appdev")
    FollowRequest.create(sender_id: other_user.id, recipient_id: user.id, status: "pending")

    visit "/#{user.username}/pending"

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

describe "/[USERNAME]/discover" do
  it "can be visited", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    visit "/#{user.username}/discover"

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

    visit "/#{user.username}/discover"

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

describe "/[USERNAME]/feed" do
  it "can be visited", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    visit "/#{user.username}/feed"

    expect(page.status_code).to be(200)
  end

  it "shows their leader's photos", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    leader = User.create(username: "leader", email: "leader@example.com", password: "appdev", private: false)
    photo = create_photo(owner: leader, caption: "leader caption")
    FollowRequest.create(sender_id: user.id, recipient_id: leader.id, status: "accepted")

    visit "/#{user.username}/feed"

    expect(page).to have_content(photo.caption)
    expect(page).to have_tag("img")
  end

  it "allows them to like their leader's photos", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    leader = User.create(username: "leader", email: "leader@example.com", password: "appdev", private: false)
    photo = create_photo(owner: leader)
    FollowRequest.create(sender_id: user.id, recipient_id: leader.id, status: "accepted")

    visit "/#{user.username}/feed"

    click_on "0 likes"

    expect(page).to have_tag("i", with: { class: ["fa-solid", "fa-heart"] })
  end

  it "allows them to un-like their leader's photos", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    leader = User.create(username: "leader", email: "leader@example.com", password: "appdev", private: false)
    photo = create_photo(owner: leader)
    FollowRequest.create(sender_id: user.id, recipient_id: leader.id, status: "accepted")
    Like.create(fan_id: user.id, photo_id: photo.id)

    visit "/#{user.username}/feed"

    click_on "1 like"

    expect(page).to have_css("i.fa-regular.fa-heart")
  end

  it "allows the user to add a comment on their leader's photos", points: 1 do
    user = User.create(username: "alice", email: "alice@example.com", password: "appdev")
    sign_in(user)

    leader = User.create(username: "leader", email: "leader@example.com", password: "appdev", private: false)
    photo = create_photo(owner: leader)
    FollowRequest.create(sender_id: user.id, recipient_id: leader.id, status: "accepted")

    visit "/#{user.username}/feed"

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

    visit "/#{user.username}/feed"

    within("#comment_#{comment.id}") do
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

    visit "/#{user.username}/feed"

    within("#comment_#{comment.id}") do
      click_on "Edit"
    end

    fill_in "comment[body]", with: "Edited comment"
    click_button "Update Comment"

    expect(page).to have_content("Edited comment")
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
