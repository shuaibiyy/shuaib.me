+++
date = "2015-11-24T23:20:37+08:00"
description = ""
title = "On Testing React Components"

+++

Recently, I tried to figure out unit testing React components. I stumbled upon a couple of good articles by [Simon Smith][simon] and [Marcin Grzywaczewski][marcin]. The former was mainly about shallow rendering and latter went into great depths to flesh out the numerous ways of approaching unit testing React components. Based on the [recommendation][react-discuss] of the React developers, I swallowed the shallow rendering pill.

Whatever approach you decide to go with, [Enzyme][enzyme] from Airbnb is a promising testing utility that should remedy a lot of the pains associated with querying the virtual dom. [Skin-deep][skin-deep] is a similar utility written specifically for shallow rendering.

[simon]: http://simonsmith.io/unit-testing-react-components-without-a-dom/
[marcin]: http://reactkungfu.com/2015/07/approaches-to-testing-react-components-an-overview/
[react-discuss]: https://discuss.reactjs.org/t/whats-the-prefered-way-to-test-react-js-components/26/2
[enzyme]: https://github.com/airbnb/enzyme
[skin-deep]: https://github.com/glenjamin/skin-deep/
