# ankiaddonconfig

This package was born out of a desire to make creating a gui config window as painless as possible. You can also use it as a shorthand to manage the config as well.

## Installing with Git Submodule
To use this package, run the following command. (Where `.` is the directory you want to install this package in)

```bash
git submodule add https://github.com/BlueGreenMagick/ankiaddonconfig.git .
```

When you want to pull the changes made in this repo, run the below command.
```bash
git submodule update --remote --recursive
```

You may also want to instruct people that after cloning your project, they will need to run the following line.
```bash
git submodule update --init
```
## Creating a custom config window

```python
from ankiaddon import ConfigManager, ConfigWindow

conf = ConfigManager()

def general_tab(conf_window: ConfigWindow) -> None:
    tab = conf_window.add_tab("General")
    tab.text("Addon Config")
    tab.checkbox("use_fruit", "Should this addon use a fruit?")
    
    fruit_labels = ["Apples", "Pears", "Grapes"] # Shown to the user in the config window
    fruit_values = ["apple", "pear", "grape"] # Actual value the json config will have
    tab.dropdown("fruit", fruit_labels, fruit_values)

    tab.color_input("apple.color", "Color of the apple:")

    # This adds a stretchable blank space.
    # If you are not sure what this does,
    # Try resizing the config window without this line
    tab.stretch() 

conf.use_custom_window()
conf.add_config_tab(general_tab)
```

When the user opens the config window, a ConfigWindow object is created. Then before it is shown, every function you registered with `conf.add_config_tab` is run. 

Each widget is linked to a single config entry. When the user interacts with a widget and saves it, its corresponding config entry is modified and saved. The config entry key that it will be linked to is passed as the first argument to the input widget. When you have a dictionary inside your config, you can link a config widget to one of its value using `"dict_name.dict_key"`. 

### Methods in ConfigLayout
When you call `ConfigWindow.add_tab(name)`, you get a ConfigLayout object.
Creating the widgets is done in ConfigLayout. All the below methods are methods of the ConfigLayout.


List of all inputs.

```python
def checkbox(self, key: str, description: str = "") -> QCheckBox:
    assert isinstance(conf[key], bool)
def dropdown(self, key: str, labels: list, values: list, description: Optional[str] = None) -> QComboBox:
    assert conf[key] in values
def text_input(self, key: str, description: Optional[str] = None) -> QLineEdit:
    assert isinstance(conf[key], str)
def number_input(self, key: str, description: Optional[str] = None,
                     minimum: int = 0, maximum: int = 99, step: int = 1,
                     decimal: bool = False, precision: int = 2) -> QSpinBox | QDoubleSpinBox:
    if decimal:
        assert isinstance(conf[key], int | bool)
    else:
        assert isinstance(conf[key], int)
def color_input(self, key: str, description: Optional[str] = None) -> QPushButton:
    assert conf[key] is 'a hex color string like "#000", "#000000" that QColor can understand'
def path_input(self, key: str, description: Optional[str] = None, get_directory: bool = False, filter="Any files (*)") 
-> Tuple[QLineEdit, QPushButton]:
    assert isinstance(conf[key], str)
```

List of all widgets:
```python
def text(self, text: str, bold: bool = False, html: bool = False, size: int = 0, multiline: bool = False) -> QLabel:
    # Text label. `size`: font size
def space(self, space: int = 1) -> None:
    # Space between widgets
def stretch(self, factor: int = 0) -> None:
    # Stretch spacing for when window resizes
```

List of all layouts:
```python
def hlayout(self) -> "ConfigLayout":
    # Horizontal layout
def vlayout(self) -> "ConfigLayout":
    # Vertical layout
def scroll_layout(self, horizontal: bool = True, vertical: bool = True) -> "ConfigLayout":
    # Scrollable layout
```
## Using ConfigManager
```python
from .ankiaddon import ConfigManager
conf = ConfigManager()

fruit = conf["fruit"]

conf["fruit"] = "apple"
conf.save() # Save conf to disk
```

If you have a dictionary in your config, you can also do this:
```python
value = conf.get("apple.color", "#ff0000") # conf["apple.color"] will raise KeyError if it doesn't exist
conf["apple.color"] = "#ffff00"

apple_color = conf.pop("apple.color")
del conf["apple.size"]
```

Other features:
```python
conf.load() # discards current config and loads config from disk.
conf.get_default("fruit") # returns the value set in config.json
conf.to_json() # returns a json copy of the config
conf.clone() # returns a deepcopy of the config dictionary
```
