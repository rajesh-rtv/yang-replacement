---
###
# Internet-Draft Markdown Template
#
# Rename this file from draft-todo-yourname-protocol.md to get started.
# Draft name format is "draft-<yourname>-<workgroup>-<name>.md".
#
# For initial setup, you only need to edit the first block of fields.
# Only "title" needs to be changed; delete "abbrev" if your title is short.
# Any other content can be edited, but be careful not to introduce errors.
# Some fields will be set automatically during setup if they are unchanged.
#
# Don't include "-00" or "-latest" in the filename.
# Labels in the form draft-<yourname>-<workgroup>-<name>-latest are used by
# the tools to refer to the current version; see "docname" for example.
#
# This template uses kramdown-rfc: https://github.com/cabo/kramdown-rfc
# You can replace the entire file if you prefer a different format.
# Change the file extension to match the format (.xml for XML, etc...)
#
###
title: "Enhancements to the YANG Language for Capturing Subtree Replacements"
abbrev: "YANG Subtree Replacements"
category: info

docname: draft-rtv-netmod-yang-subtree-replacement-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: OPS
workgroup: NETMOD Working Group
keyword:
 - next generation
 - unicorn
 - sparkling distributed ledger
venue:
  group: WG
  type: Working Group
  mail: WG@example.com
  arch: https://example.com/WG
  github: rajesh-rtv/yang-replacement
  latest: https://example.com/LATEST

author:
 -
    fullname: Rajesh TV
    organization: Cisco Systems
    email: rtv-cisco@cisco.com

normative:

informative:

...

--- abstract

YANG is a data modeling language used to define the configuration and operational data of network devices. As network technologies evolve, some nodes within YANG models may become deprecated or obsolete. This document proposes the introduction of a new mechanism using a YANG extension to specify the updated XPath node when an existing node is deprecated or obsolete. By embedding replacement information directly within the YANG model, this proposal enables programmatic inference of replacements and automated tooling for managing node replacements.


--- middle

# Introduction

YANG is a data modeling language used to define the configuration and operational data of network devices. As network technologies evolve, some nodes within YANG models may become deprecated or obsolete. This document proposes the introduction of a new mechanism using a YANG extension to specify the updated XPath node when an existing node is deprecated or obsolete. By embedding replacement information directly within the YANG model, this proposal enables a programmatic inference of replacements and enabling automated tooling for managing node replacements.

# Problem Statement

Currently, the only way to communicate replacement paths for deprecated nodes is through separate, unstructured documents like CSV files or emails. This approach suffers from several drawbacks:

- It is inefficient, requiring manual searching and correlation of information.
- It is error-prone, as the external document may not accurately reflect the node structure and paths defined in the YANG files.
- It hinders automation, as there is no standardized way for tools to programmatically identify and report replacement nodes.

# Solution

To address the challenges associated with deprecated XPath nodes in YANG models, we propose leveraging an extension-based approach rather than introducing a new core keyword to the YANG language. Specifically, we introduce a custom extension, `cisco-ext:replacement-info`, which provides metadata indicating the appropriate replacement for a deprecated or obsolete node. This extension will carry information such as the XPath or reference to the updated node, and optionally indicate whether the feature is scheduled for removal.

This approach maintains backward compatibility and minimizes disruption to existing tooling and workflows. It enables YANG developers and tools to manage deprecations more effectively, offering clear guidance for transitions and supporting long-term model evolution with improved documentation and automation support.

# Implementation of the Replacement Keyword Across Various Scenarios

## Reasoning for Path Selection

- **REPLACEMENT_ABSOLUTE_PATH:** Used when the destination leaf is in a straightforward structure, such as a container, allowing easy pinpointing from the yang root. Indicates that the XPath provided is an absolute path from the root of the YANG model.
- **REPLACEMENT_REL_PATH:** Used when the destination leaf is part of a grouping structure that might be utilized in multiple locations. This choice allows the path to fit multiple locations attributes seamlessly. Indicates that the XPath provided is a relative path from the top level of a grouping.
- **Absolute XPath (abs_path):** For nodes not within groupings, use an absolute XPath.

## Case 1: Simple Node with Replacement

- **Description:** Node that is deprecated and has a replacement node
- **Syntax:** `cisco-ext:replacement-info "REPLACEMENT_ABSOLUTE_PATH:/{File_name}:{abs_path}";`

## Case 2: Node Deprecated with No Replacement

- **Description:** Node that is deprecated and has no replacement node.
- **Syntax:** `cisco-ext:replacement-info "REPLACEMENT_ABSOLUTE_PATH:None";`

## Case 3: Grouping Cases

Identifying XPath nodes within YANG models, particularly when dealing with groupings, presents a challenge. Nodes that are imported through grouping may not be easily pinpointed using absolute XPath. The proposed solution involves uniquely identifying the grouping first using the file name and grouping name, then providing a relative XPath from the top level of the grouping. In all other cases, an absolute XPath can be provided.

- **Uniquely Identify the Grouping:** Utilize the file name and grouping name to uniquely identify the grouping within the YANG model.
- **Relative XPath for Groupings (rel_path_inside_grouping):** Provide a relative XPath from the top level of the grouping to pinpoint the node.

This approach ensures that nodes within groupings can be accurately identified and referenced, while maintaining clarity and precision for nodes outside of groupings.

### Sub-Case 1: Node Deprecated Outside Grouping, Replacement Inside Grouping

- **Description:** A node that is deprecated outside a grouping structure but has a replacement node within a specific grouping.
- **Syntax:** `cisco-ext:replacement-info "REPLACEMENT_REL_PATH:/{File_name}:{grouping_name}/{rel_path_inside_grouping}";`

### Sub-Case 2: Node Deprecated Inside Grouping, Replacement Outside in a non-group

- **Description:** A node that is deprecated within a grouping structure but has a replacement node outside any grouping.
- **Syntax:** `cisco-ext:replacement-info "REPLACEMENT_ABSOLUTE_PATH:/{File_name}:{abs_path}";`

### Sub-Case 3: Node Deprecated Inside Grouping, Replacement Inside Existing/New Grouping

- **Description:** A node that is deprecated within a grouping structure and has a replacement node within the same or a new grouping structure.
- **Syntax:** `cisco-ext:replacement-info "REPLACEMENT_REL_PATH:/{File_name}:{grouping_name}/{rel_path_inside_grouping}";`

### Sub-Case 4: Node Deprecated Outside Grouping, Replacement Outside Grouping

- **Description:** A node that is deprecated outside a grouping structure and has a replacement node also outside any grouping.
- **Syntax:** `cisco-ext:replacement-info "REPLACEMENT_ABSOLUTE_PATH:/{File_name}:{abs_path}";`

## Summary of Syntax Notations

- `/{File_name}:{grouping_name}/{rel_path_inside_grouping}`: Indicates the relative path of the replacement node within a specified grouping in a file.
- `/{File_name}:{path}`: Indicates the absolute path of the replacement node in a file.
- `None`: Indicates that there is no replacement for the deprecated node.

## Replacement Guidelines for Non-Leaf level Deprecation

When deprecating structures like container or list at their respective levels, it is essential to ensure that the replacement is explicitly mentioned at all sub-levels, including child elements such as leaf, leaf-list, or nested structures. This approach ensures clarity and provides a complete mapping of deprecated elements to their replacements, making it easier for users to transition to the new structure.

# Example Implementation

## `cisco-extensions.yang` module

```yang
module cisco-extensions {
  namespace "http://cisco.com/yang/cisco-extensions";
  prefix cisco-ext;

  organization "Cisco Systems";

  contact <mailto:cs-yang@cisco.com>;

  description
    "This module defines extensions for additional metadata.
     Copyright (c) 2024 by Cisco Systems, Inc.";

  extension replacement-info {
    argument "value";
    description
      "Provides replacement model information for deprecated/obsolete model.";
  }
}
```

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Security Considerations

This document specifies an extension to the YANG language for documenting replacement paths for deprecated or obsolete nodes. As such, it does not introduce any new security risks beyond what exists in the current YANG language specification.

Implementers should be aware that the replacement information could potentially be used for reconnaissance if it reveals information about internal system structure or capabilities not intended for public disclosure. Care should be taken to ensure that replacement path information does not inadvertently expose sensitive implementation details.

# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

The authors would like to thank the members of the NETMOD working group for their valuable input and feedback.

The authors acknowledge the original work on YANG Next Steps (https://github.com/netmod-wg/yang-next/issues/130) that provided initial inspiration for this proposal.
