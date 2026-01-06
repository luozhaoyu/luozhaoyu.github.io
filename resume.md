Creating `resume.html` to generate resume based on HTML/CSS with a modern view.

### High Level Requirements
- Review the resume grammar and spelling issues to make it smooth and concise
- Use modern HTML/CSS best practice to generate a responsive page
  - It would fully leverage modern HTML elements with correct semantics, see examples below (but not limited to these)
- CSS: It must use modern CSS frameworks for color selection and semantic HTML
  - A minimalist/class-less CSS framework is preferred
  - respect CSS framework's design decisions
    - try to reuse the component provided by the CSS framework as much as possible
    - only override when specifically needed for the requirements.
- The whole resume layout is within A4 dimension, that is easy to be exported/printed within A4
- 2-column display: left column is work experience, right column is miscellaneous info such as contact/profile summary/eduction/skill
  - Left column is the important content, which is wider than right column
- For each skill it has a progress bar to visualize the strength


### Detailed Requirements
Using picocss design with grid support, as we have 2-column display
```
<link
  rel="stylesheet"
  href="https://cdn.jsdelivr.net/npm/@picocss/pico@2/css/pico.pumpkin.min.css"
>
```


```
Content Sections:
<header>: Often used for the introductory content of a section, like the resume header with your name and contact information.
<nav>: Used for navigation links, though not as common in a single-page resume.
<main>: Encloses the main content of the resume.
<article>: Represents a self-contained composition, like a job experience entry.
<section>: Defines a thematic grouping of content.
<aside>: Contains content that is aside from the main content (e.g., skills, interests).
<footer>: Contains information about the document (e.g., last updated date). 

Text Elements:
<h1> - <h6>: Headings to structure content, with <h1> being the most important.
<p>: Paragraphs for text content.
<ul>, <ol>, <li>: Used for creating unordered and ordered lists (e.g., skills, education). 

Other Useful Elements:
<a>: Links to other pages or websites (e.g., portfolio website, LinkedIn profile).
<img>: Embeds images (e.g., your profile picture).
<div>: A generic container for grouping and styling elements.
<span>: A generic inline container for styling text.
<strong>: For bold text (e.g., job titles, key skills).
<em>: For emphasized text (e.g., important details). 


Employing semantic HTML elements (like those above) is crucial for both user experience and search engine optimization. These elements provide context and structure to your content, making it easier for screen readers to interpret and for search engines to crawl. Additionally, consider:

ARIA attributes: Use ARIA roles like role="navigation" or role="main" to further enhance the meaning of elements for assistive technologies.
Descriptive links: Instead of "Click Here," use descriptive text for your links (e.g., "View My Portfolio").
Proper alt text: Provide meaningful descriptions for images using the alt attribute. 
```