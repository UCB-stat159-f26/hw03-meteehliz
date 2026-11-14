# tennishw

A Python package that downloads a tennis player's career matches from
[tennisabstract.com](https://www.tennisabstract.com) and plots how the
player's age compares with their opponents' ages over a career, like the
New York Times Serena Williams graphic (without annotations).

## Installation

```bash
pip install git+https://github.com/UCB-stat159-f26/hw03-meteehliz.git
```

## Usage

```python
import tennishw

# Serena Williams data and plot
serena = tennishw.get_career_data("serena williams")
tennishw.plot_age_gap(serena, title="Serena Williams")

# Rafael Nadal data and plot
rafa = tennishw.get_career_data("Rafael Nadal")
tennishw.plot_age_gap(rafa, title="Rafael Nadal")

# Unknown player: raises a meaningful error
tennishw.get_career_data("GastonSanchez")
# ValueError: No player named 'GastonSanchez' was found on tennisabstract.com. ...
```

In a script (outside Jupyter), add `import matplotlib.pyplot as plt` and
`plt.show()` to display the plot.

## Public functions

| Function | Input | Output |
|---|---|---|
| `get_career_data(player_name, player_dob=None)` | Player name, e.g. `"serena williams"`, `"Serena Williams"`, `"SerenaWilliams"` | pandas DataFrame, one row per match |
| `plot_age_gap(career_df, title=None, ax=None)` | DataFrame from `get_career_data()` | matplotlib Axes with the scatter plot |

Columns of the data frame:

| Column | Meaning |
|---|---|
| `date` | Match date (tournament start date) |
| `tournament`, `surface` | Event name and court surface |
| `result` | `"W"` or `"L"` for the player |
| `opponent` | Opponent's name |
| `player_age` | Player's age in years at the match |
| `opponent_age` | Opponent's age in years at the match |
| `age_diff` | `player_age - opponent_age` (positive = opponent was younger) |

## How the data are processed

- Match data come from `https://www.tennisabstract.com/jsmatches/<FirstLast>Career.js`,
  in the variable `morematchmx`.
- **Female** files give each opponent's date of birth (`YYYYMMDD`), so the
  opponent's age is computed from it. **Male** files give the opponent's age
  directly. The package detects which format it is reading.
- The player's own date of birth is read from their tennisabstract.com
  profile page. If that ever fails, pass it yourself:
  `get_career_data("Rafael Nadal", player_dob="1986-06-03")`.

## Package structure

```
src/tennishw/
├── __init__.py   # exposes the two public functions
├── _helper.py    # private, network-free parsing/cleaning helpers
├── data.py       # get_career_data() + private download helpers
└── plot.py       # plot_age_gap() + private styling helpers
tests/
├── fake_data/    # mock FemalePlayerCareer.js and MalePlayerCareer.js
└── test_helper.py
```

## Running the tests

The tests never contact tennisabstract.com. They run on the fake JS files
in `tests/fake_data/`.

```bash
pip install -e .
pip install pytest
pytest
```

## Reflections

In HW2, my code only worked for Serena Williams, with her name and birth date built directly into the notebook. Turning it into a package meant rewriting it so that any player’s name could go in and the same clean data frame and plot would come out. The 20-line limit forced me to split the work into small functions, each doing one thing: formatting the name, extracting the matches, computing ages, and building the data frame. At first this felt slower than writing one big block of code, but it made testing much easier, because I could check each piece on its own and know exactly which step broke when something went wrong. I also learned the difference between public and private functions. Users only see get_career_data() and plot_age_gap(), while the helper functions starting with _ do the work behind the scenes. This keeps the package simple to use, since someone only needs to learn two functions, and it lets me change the internals later without breaking anyone’s code.

I didn’t expect the Career JS files to differ by gender. Women’s files store each opponent’s date of birth, while men’s files store the opponent’s age directly. This meant the same column had to be handled two different ways depending on the player. I handled it by checking the format of that column: if every value is an 8-digit date, it’s a female file, so the opponent’s age is computed as match date minus birth date; otherwise the age is used as is. It taught me not to assume that data from the same website is always structured the same way, and to look at the raw data before writing code.
