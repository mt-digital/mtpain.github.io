---
title: Python unittesting and code coverage, all with nosetests
tags: coding programming
layout: post
date: 2015-01-08 
---

### coverage.py?

I tried to google something like `code coverage in python`, but the first 
results were for [coverage.py](http://nedbatchelder.com/code/coverage/). 
The tutorial seemed to be assuming some knowledge that I didn't have. 

## nosetests

I've been using nosetests exclusively for testing python code, mainly because
of the ability to check if exceptions are raised, like so:

{% highlight python %}
@raises(HTTPError)
def test_client_auth_fail_on_bad_ip(self):
    """ Test that failed authorization is correctly caught """
    bad_ip = "129.24.."
    Client(bad_ip, "fake_user", "fake_passwd")
{% endhighlight %}

It has a few options and is a fine unittesting framework. `nose` makes it 
easy to have testing not only be an efficiency booster because it enables
test driven development, but also a means of documenting code capabilities as a
test-driven approach should do.

Here is how I use `nose` on its own, without generating a code coverage report.
First, I write my tests in this way, with informative docstrings. The 
`nosetests` command takes an argument we'll use to print verbosely. It will
print the first line of the docstring, so here we will provide a short but 
informative description of what the test is doing. I keep all my tests in a
`test/` subfolder which is automatically discovered by `nosetests`. The 
`setUp` method is run first: here you can initialize any variables needed by
multiple tests. The tests themselves are appropriately-named (start with 
`test_`) methods of the TestRobot class.

### test/test_foo.py

{% highlight python %}
"""
Testing functions from the `foo` module. 

Date: January 1, 2015
"""

from foo import Robot, Dance, Song, load_dance_from_file, 

class TestRobot(unittest.TestCase):
    """
    We must make sure the robot can dance and sing
    """
    def setUp(self, ...):
        # init variables here
        self.expected_dance = load_dance_from_file("dancing.dnc")
        self.expected_song = load_song_from_file("singing.dnc")
        # use a standard robot and a tall robot
        self.robot = Robot()
        self.tall_robot = Robot(height_level="tall")

    def test_dance_from_steps(self):
        """
        Given five steps in a Steps object, check the robot properly learns the dance
        """
        steps = Steps(["L", "R", "UP", "UP", "DN"])
        calculated_dance = self.robot.learn_dance_from_steps(steps)
        # I haven't found convenience equality operators too convenient
        assert calculated_dance == self.expected_dance

    @raises(TypeError)
    def test_load_song_into_dance(self):
        """
        Check that when a Song object is passed to the learn_dance function, a TypeError is thrown
        """ 
        # if this raises a TypeError, then this test passes
        song = Song(["DO", "RE", "DO", "MI"])
        dance = self.robot.learn_dance_from_steps(song)
{% endhighlight %}

Then to get a verbose report of the tests that are running and whether or not
the test passed, use

{% highlight bash %}
nosetests -v
{% endhighlight %}

## Seeing how well-covered your tests actually are

Getting a nicely-printed report is shockingly easy. Just follow the next two
steps:

1. `nosetests -v --with-coverage`
2. `coverage report -m`

You will see a report like this, taken from a current project, the 
[wc-wave adaptors](https://github.com/tri-state-epscor/adaptors):

{% highlight bash %}
Name                            Stmts   Miss  Cover   Missing
-------------------------------------------------------------
adaptors/__init__                   0      0   100%
adaptors/isnobal                  321    243    24%   67-80, 98-169, 175-177, 183-187, 193-204, 217, 232, 273-281, 313, 319-322, 329, 336-337, 340, 368-407, 414-420, 427-438, 448-562, 573-576, 584, 592-644, 652-675, 685-695, 709-730, 733, 747-756
adaptors/test/test_vw_adaptor     110     11    90%   26-38, 226, 248
adaptors/watershed                138      9    93%   28-29, 135, 320-324, 351, 379
-------------------------------------------------------------
TOTAL                             569    263    54%
{% endhighlight %}

## Conclusion

So there you have it, code coverage reports with one extra option and one extra
bash command on top of what you normally do when you test with nosetests!
