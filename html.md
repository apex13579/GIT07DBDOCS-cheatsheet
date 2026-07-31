# Notes
- Html is the structure of a page
- you can see the html of any element by right clicking on it and choosing to inspect.
- the program used to write is a code editor
- online code editors are codepen.io or replit.com and vs code. I think in my methodology I will like Joplin with plugins and gitea to push to GitHub
- web browsers look for your main file to be index.html
- an element is an individual component of html
- most elements are container elements meaning they hold onto whatever is in between the opening and closing tags
- !DOCTYPE html is what all html files are going to start with
- html elements are often nested inside other html elements. it does not affect how the web page renders but it does make it easier to read, and it is good practice to indent all nested elements for clean code
- attributes effect how the html behaves. it is always followed by a value that will be in "". the order doesn't matter as long as there is a space between each attribute and element
  - src=source. is followed by ="website URL"
  - alt=alternative. a written description of the photo in ""
  - href= links to external sites. a # fill keep it a link, but it will no longer link to anything
  - target="_blank". opens in a new tab
  - action= tells the code what action to take
  -  required=makes something required in order to interact. most have the need for an = but this is stand alone
  -  id=used to identify specific elements
  -  for=allows assisting technology to create a linked relationship to the parent and child input element
  -  type=what kind of element are you wnating to deal with
  -  checked=to make a checkbox be checked unless otherwise. this is an attribute with no value
  -  class="lists"
  -  rel=used to specify the relationship between the linked resource and the HTML document.
  -  
- in the code if you copy and paste the anchor and href line where a word would be it turns those words into a clickable link. can be done with photos as well.
- as long as you have an image in the same folder as your html file you can link to it by putting the file name instead of a URL.
- all radio buttons can be put in a radio button group ensuring that only one answer is selected
- line breaks have no effect on how a browser renders a page. to add a line break use the br tag
- div elements are often used as target with css using a class attribute
- It is considered best practice to separate your HTML and CSS in different files.
- 

| elements | purpose | why it matters |
|---|---|---|
|<>|opening tag|marks the beginning of what you are doing|
|</>|closing tag|marks the end of what you are doing|
|h|signifies a heading|There are six heading elements in HTML: h1 through h6. They're used to show the importance of sections on your webpage, with h1 being the most important and h6 the least.|
|p|signifies a paragraph|it signifies where the paragraphs are
|<!---|starts comments|notation|
|--->|ends comments|notation|
|img|image tag|a space for images|
|a|anchor|used to create a link you can click on|
|#|placeholder|placeholder. also used to change the behavior of a link using javascript.|
|ul|unordered list|a list that is unordered|
|li|list item|list item|
|ol|ordered list|ordered list. will be numbered|
|strong|enboldens text|edits the text to be enboldened|
|em|emphosise|ittalicizes text|
|form|establishes a form|allows user to embed a form for various things|
|input|inputs data|inputs data based on the type specified. does not need closing tags|
|button|a clickable button|adds a clickable button to the website. this does need a closing tag and anything in between the two is what will be on the button|
|label|makes the input a button|makes the words of the input into a button to click on|
|br|line break|makes a line break in the code of the web page to render|
|div|division|the most commonly used element of all and a general purpose container for other elements|
|footer|creates another segment|creates another segment for the purpose of organization. does nothing visually|
|head|creates a segment|like the footer but you can also include the java and css files for style|
|link|used to link to external resources|is used to link to external resources like stylesheets and site icons|








# template
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>CatPhotoApp</title>
  </head>
  <body>
    <h2>CatPhotoPP</h2>
    <main>
      <h3>Cat Photos</h3>
      <!--- TODO: Add link to view more cat photos.
      <p>Click here to view more <a href="#" target="_blank">Cat photos</a>.</p>
      <img src="https://fcc.im/fcc-relaxing-cat" alt="A cute orange cat lying on its back.">
      <h3>Cat Lists</h3>
      <div class="lists">
        <p>Things cats <em>love</em>:</p>
        <ul>
          <li>Cat nip</li>
          <li>Laser pointers</li>
          <li>Lasagna</li>
        </ul>
        <img src="lasagna.jpg" alt="lasagna">
        <p>Things cats <strong>hate</strong>:</p>
        <ol>
          <li>Flea treatment</li>
          <li>thunder</li>
          <li>other cats</li>
        </ol>
      </div>
      <img src="cats.jpg" alt="Cats">
      <form action="/submit-cat-photo">
        <label for="indoor"><input id="indoor" type="radio" name="indoor-outdoor" checked> indoor</label><br>
        <label for="output"><input id="outdoor" type="radio" name="indoor-outdoor"> outdoor</label><br>
      
        <label for="loving"><input id="loving" type="checkbox" name="personality" checked>loving</label><br>
        <label for="loving"><input id="lazy" type="checkbox" name="personality">loving</label><br>
        <label for="loving"><input id="energetic" type="checkbox" name="personality">loving</label><br>
        
        <input type="text" placeholder="cat photo url" required><br>
        <button type="submit">Submit</button>
      </form>
    </main>
    <footer>
      <p><small>No Copyright - <a href="https://freeCodeCamp.org">freeCodeCamp.org</a></small></p>
    </footer>
  </body>
</html>
