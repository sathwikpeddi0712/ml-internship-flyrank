# Explain It Like You Built It: Demystifying the Portfolio Build
**Intern:** Sathwik Peddi | **Track:** General AI Fluency (ML Focus) | **Assignment:** Week 6 - Explain It Like You Built It

---

When building a modern web application or portfolio, it’s easy to write code that "just works" without fully understanding the underlying mechanics. Below is a plain-words explanation of two core parts of my build: **how the layout adapts to different screens (CSS Grid)** and **how saving files on my computer makes them live on the internet (GitHub Pages)**. 

Imagine explaining this to a friend who has never written a single line of code!

---

## 📱 1. How the Layout Adapts to Phones vs. Laptops (CSS Grid & Media Queries)

Have you ever visited a website on your phone, and the text was so tiny you had to pinch and zoom just to read it? That happens because the page's layout is rigid. To prevent this, my portfolio uses a CSS layout tool called **CSS Grid** combined with **Media Queries**.

### The Metaphor: The Grid as a Modular Bookshelf
Imagine a modular bookshelf. 
* On a wide wall (a desktop or laptop screen), we have plenty of horizontal space. We can place a wide TV screen and a narrow stack of books side-by-side.
* On a narrow wall (a smartphone screen), we don't have enough horizontal space to put them side-by-side. We have to stack the books *underneath* the TV screen.

### The Code in Plain English
In the stylesheet for the Case Study page (`docs/case-study.html`), we define this container layout:

```css
.main-layout {
    display: grid;
    grid-template-columns: 2.3fr 0.7fr;
    gap: 48px;
}
```

* **`display: grid`**: This tells the browser, *"Treat this section of the webpage like our modular bookshelf."*
* **`grid-template-columns: 2.3fr 0.7fr`**: Instead of locking the sizes in inches or pixels, we use **Fractional Units (`fr`)**. This tells the browser to split the screen into 3 total parts (2.3 + 0.7). The main content gets a wide `2.3` shares of the screen (about 77%), and the sidebar gets a narrow `0.7` shares (about 23%).
* **`gap: 48px`**: This leaves a clean, breathable 48-pixel gap between the columns.

But what happens when the screen gets narrow? We use a **Media Query**, which is like a conditional trigger:

```css
@media (max-width: 768px) {
    .main-layout {
        grid-template-columns: 1fr;
        gap: 40px;
    }
}
```

* **`@media (max-width: 768px)`**: This is the browser checking: *"Is the screen narrower than 768 pixels (the size of a typical tablet or phone)?"*
* **`grid-template-columns: 1fr`**: If the answer is yes, the browser drops the two-column layout and switches to `1fr`. This means: *"Make the shelf only one column wide. Let the main content take up 100% of the width, and automatically push the sidebar content down underneath it."*

Because of this simple switch, the site looks premium and readable whether you're viewing it on an ultra-wide monitor or a mobile phone!

---

## 🚀 2. How Deploying Pushes the Site Live (GitHub Pages from `/docs`)

When you write HTML and CSS files on your computer, they are just documents saved on your local hard drive. If you send someone the link `file:///C:/Users/sathw/index.html`, they can't open it because that file only exists on *your* physical machine. 

To make it public, we need to **deploy** it. Here is how my build does that automatically using **GitHub Pages**.

### The Metaphor: The Public Bulletin Board
Imagine you've written a beautiful newsletter (your HTML files) on your personal laptop. To let the whole town read it, you need to copy it onto a public bulletin board that anyone can walk up to. 

* **Git** is the tracking system that logs every edit you make to your newsletter.
* **GitHub** is the public community center where your bulletin board is located.
* **GitHub Pages** is a special service that reads files from a designated folder on your bulletin board and turns them into a public website address that browsers can read.

### How the Deploy Pipeline Works Step-by-Step
In my project, the code for the portfolio is placed in a folder called `/docs` inside my GitHub repository (`ml-internship-flyrank`).

1. **Local Save:** I write or modify `index.html` and `case-study.html` inside the `/docs` folder on my computer.
2. **Git Commit & Push (Delivering to the Center):** I package these changes into a "commit" (a timestamped snapshot) and run a command to "push" it to GitHub. This copies the files from my local hard drive to GitHub's cloud servers.
3. **The `/docs` Detection:** GitHub Pages is configured to watch the `/docs` folder of my repository. The moment it sees new files arrive in that folder, it triggers an automated build runner.
4. **Instant Publishing:** Within seconds, GitHub Pages makes those files accessible at a live web URL: 
   👉 **`https://sathwikpeddi0712.github.io/ml-internship-flyrank/`**

By using the `/docs` folder structure inside the main repository, I don't need to rent separate web servers, configure complex hosting accounts, or pay monthly fees. GitHub handles all the heavy lifting, serving the files instantly to anyone who clicks the link.
