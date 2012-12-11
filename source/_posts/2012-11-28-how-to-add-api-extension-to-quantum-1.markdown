---
layout: post
title: "how to add api extension to quantum 1"
date: 2012-11-28 11:10
comments: true
categories: 
---

未完成！！！
<!--more-->

{}
class ExtensionManager(object):
    """Load extensions from the configured extension path.

    See tests/unit/extensions/foxinsocks.py for an
    example extension implementation.

    """
    def __init__(self, path):
        LOG.info(_('Initializing extension manager.'))
        self.path = path 
        self.extensions = {}
        self._load_all_extensions()

{}

class ResourceExtension(object):
    """Add top level resources to the OpenStack API in Quantum."""

    def __init__(self, collection, controller, parent=None,
                 collection_actions={}, member_actions={}):
        self.collection = collection
        self.controller = controller
        self.parent = parent
        self.collection_actions = collection_actions
        self.member_actions = member_actions


