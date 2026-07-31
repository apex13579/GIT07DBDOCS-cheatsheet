# Notes
- Html is the structure of a page
- you can see the html of any element by right clicking on it and choosing inspect.
- the program used to write is a code editor
- online code editors are codepen.io or replit.com and vs code. I think in my methodology I will like joplin with plugins and gitea to push to github
- web browsers look for your main file to be index.html
- an element is an individual component of html
- most elements are container elements meaning they hold onto whatever is in between the opening and closing tags
- !DOCTYPE html is what all html files are going to start with
- html elements are often nested inside other html elements. it does not effect how the web page renders but it does make it easier to read and it is good practice to indent all nested elements for clean code
- atteibutes effect how the html behaves. it is always followed by a value that will be in ""
  - src=source. is followed by ="website url"
  - alt=alternative. a written description of the photo in ""
  - href= links to external sites
- 

| elements | purpose | why it matters |
|---|---|---|
|<>|opening tag|marks the begining of what you are doing|
|</>|closing tag|marks the end of what you are doing|
|h|signifies a heading|There are six heading elements in HTML: h1 through h6. They're used to show the importance of sections on your webpage, with h1 being the most important and h6 the least.|
|p|signifies a paragraph|it signifies where the paragraphs are
|<!---|starts comments|notation|
|--->|ends comments|notation|
|img|image tag|a space for images|
|a|anchor|used to create a link you can click on|



# template
<!DOCTYPE html>
<html>
  <body>
    <h2>CatPhotoPP</h2>
    <main>
      <h3>Cat Photos</h3>
      <!--- TODO: Add link to view more cat photos.
      <p>Click here to view more cat photos.
      </p>
      <a href="catphotos.com>Cat photos</a>
      <img src="https://fcc.im/fcc-relaxing-cat" alt="A cute orange cat lying on its back.">
    </main>  
  </body>
</html>
