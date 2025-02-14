---
title: 
author: 
publishedAt: <% moment().toISOString() %>
draft: false
mode: 
description: 
preview: 
keywords:
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

if (category === "\_") {
	const category = await tp.system.prompt("new Category");
	await tp.file.move(`/${parentFolderPath}/${category}/${title}`);
} else {
	await tp.file.move(`${category}/${title}`);
}
await tp.file.rename(title);
%>