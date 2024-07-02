
# XTS Base Code

![XTS Base](/Assets/TrackDigital.png)


## Quick Start

- [Download the Latest Release](https://github.com/Beckhoff-USA-Community/XTS_Base/releases/latest)


## Project Links

- [Project Documentation](https://beckhoff-usa-community.github.io/XTS_Base/)
- [Beckhoff XTS](https://www.beckhoff.com/en-us/products/motion/xts-linear-product-transport/)
- [XTS Infosys](https://infosys.beckhoff.com/english.php?content=../content/1033/driveinfosys/9921860875.html&id=)


## Version 2.x

Version 2.x is the result of years of deploying this code on production equipment throughout the world. As XTS was used in more and more applications, and as the underlying XTS libraries gained capabilities the 1.x branch added these capabilities. This caused the 1.x code to grow substantially, including a lot of boilerplate code in MAIN.tcPOU.

The 2.x revision overhauls this boilerplate code and moves the majority of it into a new object called just once in MAIN. This also provides the ability to more easily act on groups of movers and adds additional features to movers. The interface to Movers, Stations, PositionTriggers and other objects remains nearly identical, if not the same to reduce any learning curve in transitioning to this version. Finally, by removing boilerplate code we streamline the user facing side of the XTS controls and make it more straight-forward as to where to begin coding your application.

### Upgrade recommendations

Because of the nature of equipment in industrial facilities, validation cycles and uptime considerations we only recommend using Version 2.x on new projects.


## Development

You can contribute by:
- Submitting [bug reports](https://github.com/Beckhoff-USA-Community/XTS_Base/issues)
- Submitting [pull requests](https://github.com/Beckhoff-USA-Community/XTS_Base/pulls)
- Joining the discussion
