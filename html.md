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
  - href= links to external sites. a # fill keep it a link, but it will no longer link to anything
  - target="_blank". opens in a new tab
  - action= tells the code what action to take
  -  required=makes something required in order to interact. most have the need for an = but this is stand alone
  -  
- in the code if you copy and past the anchor and href line where a word would be it turns those words into a clickable link. can be done with photos as well.
- as long as you have an image in the same folder as your html file you can link to it by putting the file name instead of a url.

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







# template
<!DOCTYPE html>
<html>
  <body>
    <h2>CatPhotoPP</h2>
    <main>
      <h3>Cat Photos</h3>
      <!--- TODO: Add link to view more cat photos.
      <p>Click here to view more <a href="#" target="_blank">Cat photos</a>.</p>
      <img src="https://fcc.im/fcc-relaxing-cat" alt="A cute orange cat lying on its back.">
      <h3>Cat Lists</h3>
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
      <img src="cats.jpg" alt="Cats">
      <form action="/submit-cat-photo">
        <label><input type="radio"> indoor</label>
        <label><input type="radio"> outdoor</label>
        <input type="text" placeholder="cat photo url" required>
        <button type="submit">Submit</button>
      </form>
    </main>  
  </body>
</html>
