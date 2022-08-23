# What is this?

This a static website for me, a fork of [academic pages](https://github.com/academicpages/academicpages.github.io).

Right now, I'm using it to host a little portfolio of mine and some written tutorials. The code for this website is totally open-source, feel free to copy anything that you would like to use.

## A few pointers

Here's a few pointers if you're forking Academic Pages or this website for your own use.

- All the CSS for the website is located under [_sass](/_sass/)
  - Global variables such as font sizes and colors are located in [_variables.scss](/_sass/_variables.scss)
  - Syntax highlighting CSS is [here](/_sass/_syntax.scss)
- To turn your site into dark mode:
  - Copy the [skins folder](_sass/skins/) into your project
    - Make sure you place it under the [_sass](_sass/) folder **in your project**
  - Copy **only the colors** from [_variables.scss](/_sass/_variables.scss)
  - Edit the colors in the page footer, found in [_footer.scss](/_sass/_footer.scss)
- To import and use fonts from Google Fonts:
  - Pick your fonts on the [Google Fonts](https://fonts.google.com) website
  - Take the generated `<link>` tags and place them anywhere in the [custom header file](/_includes/head/custom.html)
  - Edit the [_variables.scss](/_sass/_variables.scss) file or whatever CSS file you need to choose the right font using `font-family`

# Find me here

<p align="center">
<a href="https://www.youtube.com/channel/UCD7K_FECPHTF0z5okAVlh0g/featured" target="blank"><img src="https://img.shields.io/badge/NekotoArts-%23FF0000.svg?style=for-the-badge&logo=YouTube&logoColor=white" alt="NekotoArts" /></a>
<a href="https://twitter.com/NekotoArts" target="blank"><img src="https://img.shields.io/badge/NekotoArts-%231DA1F2.svg?style=for-the-badge&logo=Twitter&logoColor=white" alt="NekotoArts" /></a>
<a href="https://nekotoarts.itch.io/" target="blank"><img src="https://img.shields.io/badge/Itch-%23FF0B34.svg?style=for-the-badge&logo=Itch.io&logoColor=white" /></a>
<a href="https://ko-fi.com/nekoto" target="blank"><img src="https://img.shields.io/badge/Ko--fi-F16061?style=for-the-badge&logo=ko-fi&logoColor=white" /></a>
<a href="https://godotshaders.com/author/nekotoarts/" target="blank"><img src="https://img.shields.io/badge/Godot_Shaders-%23FFFFFF.svg?style=for-the-badge&logo=godot-engine" /></a>
<a href="https://reddit.com/user/XDGregory" target="blank"><img src="https://img.shields.io/badge/Reddit-FF4500?style=for-the-badge&logo=reddit&logoColor=white" /></a>
<a href="https://discord.gg/eX5Qygqve6" target="blank"><img src="https://img.shields.io/badge/NekotoArts_Server-%235865F2.svg?style=for-the-badge&logo=discord&logoColor=white" /></a>
</p>
