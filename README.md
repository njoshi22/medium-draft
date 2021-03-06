# medium-draft - [demo](http://bitwiser.in/medium-draft/)

A medium like rich text editor built upon [draft-js](https://facebook.github.io/draft-js/) with an emphasis on eliminating mouse usage by adding relevant keyboard shortcuts.

### Features

- Focus on keyboard shortcuts and auto transform of text blocks.
- Image addition with support for rich text captioning.
- Minimize mouse usage.
- Autolists.
- Proper handling of `RETURN` presses.
- It also has implementations of some custom blocks like:
    - `caption` - Can be used as a caption for media blocks like image or video instead of nested `draft-js` instances for simplicity.
    - `block-quote-caption` - Caption for `blockquote`s.
    - `todo` - Todo text with a checkbox.

##### Following are the keyboard shortcuts to toggle block types (<kbd>Alt and CTRL</kbd> for Windows/Linux and <kbd>Option and Command</kbd> for OSX)
*   <kbd>Alt/Option</kbd> +

    *   <kbd>1</kbd> - Toggle Ordered list item
    *   <kbd>*</kbd> - Toggle Unordered list item
    *   <kbd>@</kbd> - Add link to selected text.
    *   <kbd>#</kbd> - Toggle Header-three.
    *   <kbd><</kbd> - Toggle Caption block.
    *   <kbd>></kbd> - Toggle unstyled or paragraph block.
    *   <kbd>H</kbd> - Highlight selection.

##### Editor level commands

These commands are not a part of the core editor but have been implemented in the example code that uses the `medium-draft` editor.

*   <kbd>Command/CTRL</kbd> + <kbd>S</kbd> - Save current data to `localstorage`.
*   <kbd>Alt + Shift</kbd> + <kbd>L</kbd> - Load previously saved data from `localstorage`.

##### Special characters while typing: While typing in an empty block, if the content matches one of the following, that particular block's type and look will be changed to the corresponding block specified below

*   `--` - If current block is `blockquote`, it will be changed to `block-quote-caption`, else `caption`.
*   `*.` `(An asterisk and a period)` - `unordered-list-item`.
*   `*<SPACE>` `(An asterisk and a space)` - `unordered-list-item`.
*   `-<SPACE>` `(A hyphen and a space)` - `unordered-list-item`.
*   `1.` `(The number 1 and a period)` - `unordered-list-item`.
*   `##` - `header-two`.
*   `[]` - `todo`.
*   `==` - `unstyled`.

### Installation

- **npm**.
    - `npm install medium-draft`.
    - `import Editor from 'medium-draft'`
- **Browser**
    - Include `<link rel="stylesheet" type="text/css" href="https://unpkg.com/medium-draft/dist/medium-draft.css">` in `<head>`
    - Include `<script src="https://unpkg.com/medium-draft/dist/medium-draft.js"></script>`. **medium-draft** is available in the global object as `MediumDraft`.

### Usage

`medium-draft` sits on top of `draft-js` with some built in functionalities and blocks. Its API is almost the same as that of `draft-js`. You can take a look at [the demo editor's code](https://github.com/brijeshb42/medium-draft/tree/master/src/example.js) to see the implementation.

At the minimum, you need to provide `editorState` and `onChange` props, the same as `draft-js`.

```javascript

import React from 'react';
import ReactDOM from 'react-dom';

import {
  Editor,
  createEditorState,
} from 'medium-draft';

class App extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      editorState: createEditorState(), // for empty content
    };

    /*
    this.state = {
      editorState: createEditorState(data), // with content
    };
    */

    this.onChange = (editorState) => {
      this.setState({ editorState });
    };
  }

  componentDidMount() {
    this.refs.editor.focus();
  }

  render() {
    const { editorState } = this.state;
    return (
      <Editor
        ref="editor"
        editorState={editorState}
        onChange={this.onChange} />
    );
  }
};

ReactDOM.render(
  <App />,
  document.getElementById('app')
);
```

### Customizing side buttons

`medium-draft`'s `Editor` accepts a prop called `sideButtons`. By default, there is only one (image) button, but you can add more. The `sideButtons` prop must be an array of objects with each object having the following signature:

```js
{
  "title": "unique-button-name",
  "component": ButtonComponent
}
```

For ex:

```js
{
  "title": "Image",
  "component": ImageSideButton
}
```

Example code:

Right now, the image button simply adds an image inside the editor using `URL.createObjectURL`. But if you would like to first upload the image to your server and then add that image to the editor, you can follow one of the 2 methods:

1. Either extend the default `ImageSideButton` component that comes with `medium-draft`.

2. Or create your own component with the complete functionality yourself.

For simplicity, we will follow the first method. If you study the [implementation](src/components/sides/image.js) of `ImageSideButton`, you will see an `onChange` method that receives the file chooser event where the seleced files are available as `event.target.files`. We will simply override this method as we don't want to customize anything else. Also note that each side button component receives `getEditorState` function (returns the draft `editorState`), `setEditorState(newEditorState)` function (sets the new editorState) and `close` function which you need to call manually to close the side buttons list:

```javascript
import React from 'react';
import {
  ImageSideButton,
  Block,
  addNewBlock,
  createEditorState,
  Editor,
} from 'medium-draft';
import 'isomorphic-fetch';

class CustomImageSideButton extends ImageSideButton {
  
  /*
  We will only check for first file and also whether
  it is an image or not.
  */
  onChange(e) {
    const file = e.target.files[0];
    if (file.type.indexOf('image/') === 0) {
      // This is a post request to server endpoint with image as `image`
      const formData = new FormData();
      formData.append('image', file);
      fetch('/your-server-endpoint', {
        method: 'POST',
        body: formData,
      }).then((response) => {
        if (response.status === 200) {
          // Assuming server responds with
          // `{ "url": "http://example-cdn.com/image.jpg"}`
          return response.json().then(data => {
            if (data.url) {
              this.props.setEditorState(addNewBlock(
                this.props.getEditorState(),
                Block.IMAGE, {
                  src,
                }
              ));
            }  
          });
        }
      });
    }
    this.props.close();
  }

}

// Now pass this component instead of default prop to Editor example above.
class App extends React.Component {
  constructor(props) {
    super(props);

    this.sideButtons = [{
      title: 'Image',
      component: CustomImageSideButton,
    }];

    this.state = {
      editorState: createEditorState(), // for empty content
    };

    /*
    this.state = {
      editorState: createEditorState(data), // with content
    };
    */

    this.onChange = (editorState) => {
      this.setState({ editorState });
    };
  }

  componentDidMount() {
    this.refs.editor.focus();
  }

  render() {
    const { editorState } = this.state;
    return (
      <Editor
        ref="editor"
        editorState={editorState}
        onChange={this.onChange}
        sideButtons={this.sideButtons}
      />
    );
  }
};
```


### Issues

- [ ] Figure out a way to show placeholder text for empty image captions.
- [x] Currently, the toolbar that appears when text is selected needs to be fixed regarding its position in the viewport.

### Developer

- Clone this repo `git clone https://github.com/brijeshb42/medium-draft.git`.
- Install node packages `npm install`.
- Start local demo `npm run dev`. This will start a local server on port `8080`.
- Build using `npm run build`.

#### LICENSE

MIT
