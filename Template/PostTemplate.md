---
title: 
date: <% moment().toISOString() %>
draft: false
mode: 
summary: 
banner: 
tags: 
updatedAt: 
tags:
---
<%*
const getSubFolders = () => {

const allFolders = app.vault
.getAllLoadedFiles()
.filter((file) => file instanceof tp.obsidian.TFolder);

const subFolders = allFolders
.map((folder) => folder.path);
    subFolders.push("_");
    return subFolders;
};

const title = await tp.system.prompt("Title");
const subFolders = getSubFolders();
const category = await tp.system.suggester(subFolders, subFolders);

if (title === null || category === null) return;

await tp.file.move(`${category}/${title}/${title}`);

await tp.file.rename(title);
%>