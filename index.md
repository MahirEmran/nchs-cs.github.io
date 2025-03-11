---
layout: default
title: "North Creek High School"
subtitle:  "Computer Science"
permalink: /index
---

![NCHS Campus](/static/NCHS-Building-1-South-20170901.jpg)

# Advanced Programming Topics

- [Final Project Directions](advanced-topics/final-project/index.md)
 
# Intermediate Data Programming

- [Final Project Directions](idp/final-project/handouts/index.md)

# Change Log

<p>Below are changes to the website. Pay attention to these to review any updates from what you have previously reviewed.</p>

<ul id="changelog"></ul>

<script>
  document.addEventListener("DOMContentLoaded", function() {
    const commitPrefix = "PUB:"; // Adjust this prefix to match your tagging format
    fetch("https://api.github.com/repos/NCHS-CS/nchs-cs.github.io/commits")
      .then(response => response.json())
      .then(data => {
        let changelog = document.getElementById("changelog");
        let filteredCommits = data.filter(commit => commit.commit.message.startsWith(commitPrefix));
        
        if (filteredCommits.length === 0) {
          changelog.innerHTML = "<li>No relevant commits found.</li>";
          return;
        }
        
        filteredCommits.forEach(commit => {
          let entry = document.createElement("li");
          entry.innerHTML = `
            <strong>${commit.commit.author.date}</strong>: 
            ${commit.commit.message.replace(commitPrefix, "").trim()} <br>
            <a href="${commit.html_url}" target="_blank">View Commit</a>
          `;
          changelog.appendChild(entry);
        });
      })
      .catch(error => {
        console.error("Error fetching commit data:", error);
        document.getElementById("changelog").innerHTML = "<li>Error loading changelog.</li>";
      });
  });
</script>
