# Sushant's Blog

This is the repository for my personal blog, powered by [Jekyll](https://jekyllrb.com/) and hosted on [GitHub Pages](https://pages.github.com/).

## Running the Blog Locally

To preview the blog on your local machine before pushing changes to GitHub, follow these steps. 

### Prerequisites (Using `rbenv`)

We recommend using `rbenv` to manage your Ruby versions.
1. Install `rbenv` and `ruby-build`:
   ```bash
   sudo apt update
   sudo apt install git curl libssl-dev libreadline-dev zlib1g-dev autoconf bison build-essential libyaml-dev libreadline-dev libncurses5-dev libffi-dev libgdbm-dev
   curl -fsSL https://github.com/rbenv/rbenv-installer/raw/HEAD/bin/rbenv-installer | bash
   ```
2. Follow the terminal instructions to add `rbenv` to your bash/zsh profile.
3. Install Ruby (e.g., version 3.2.2) and set it globally:
   ```bash
   rbenv install 3.2.2
   rbenv global 3.2.2
   ```

### Installation

1. Install modern bundler and jekyll:
   ```bash
   gem install bundler jekyll
   ```
2. Navigate to the repository and install project dependencies:
   ```bash
   cd sushant.github.io
   bundle install
   ```

### Running the Server

Start the local Jekyll server:
```bash
bundle exec jekyll serve
```
Your blog will be accessible at `http://127.0.0.1:4000/`.

---

## Writing a New Post

All blog posts are located in the `_posts/` directory.

To create a new post:
1. Create a new file inside `_posts/` following the naming convention: `YEAR-MONTH-DAY-title.markdown` (e.g., `2026-03-07-my-first-post.markdown`).
2. Add the required YAML front matter at the top of the file:
   ```yaml
   ---
   layout: post
   title: "Your Post Title Here"
   date: 2026-03-07 10:00:00 -0800
   ---
   ```
3. Write your content below the front matter using Markdown.
4. If the local server is running, the site will automatically regenerate when you save the file!
5. When ready, commit and push the new file to GitHub. GitHub Pages will build and deploy it automatically.
