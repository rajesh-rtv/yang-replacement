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
date: 2025-06-01
consensus: true
v: 3
area: OPS
workgroup: NETMOD Working Group
keyword:
 - yang
 - netconf
 - yang-models
 - deprecation
 - model-evolution
venue:
  group: NETMOD
  type: Working Group
  mail: netmod@ietf.org
  arch: https://mailarchive.ietf.org/arch/browse/netmod/
  github: rajesh-rtv/yang-replacement
  latest: https://datatracker.ietf.org/doc/draft-rtv-netmod-yang-subtree-replacement/

author:
 -
    fullname: Rajesh Tarakkad Venkateswaran
    organization: Cisco Systems
    email: rtv@cisco.com
 -
    fullname: Sai Venkata Giri Karnati
    organization: Cisco Systems
    email: saikarna@cisco.com
 -
    fullname: Sarthak Jain
    organization: Cisco Systems
    email: sarthakj@cisco.com
 -
    fullname: Veena Ramamoorthy
    organization: Cisco Systems
    email: vemoorth@cisco.com
 -
    fullname: Venkata Harish Nagamangalam
    organization: Cisco Systems
    email: vnagaman@cisco.com

normative:
  RFC7950:
    title: The YANG 1.1 Data Modeling Language
    author:
      - ins: M. Bjorklund, Ed.
        name: Martin Bjorklund
        org: Tail-f Systems
    date: 2016-08
    seriesinfo:
      RFC: 7950
      DOI: 10.17487/RFC7950
  RFC8407:
    title: Guidelines for Authors and Reviewers of Documents Containing YANG Data Models
    author:
      - ins: A. Bierman
        name: Andy Bierman
        org: YumaWorks
    date: 2018-10
    seriesinfo:
      RFC: 8407
      DOI: 10.17487/RFC8407

informative:
  RFC8340:
    title: YANG Tree Diagrams
    author:
      - ins: M. Bjorklund
        name: Martin Bjorklund
        org: Tail-f Systems
      - ins: L. Berger, Ed.
        name: Lou Berger
        org: LabN Consulting, L.L.C.
    date: 2018-03
    seriesinfo:
      RFC: 8340
      DOI: 10.17487/RFC8340

...

--- abstract

YANG is a data modeling language used to define the configuration and operational data of network devices. As network technologies evolve, some nodes within YANG models may become deprecated or obsolete. This document proposes the introduction of a new mechanism using a YANG extension to specify the updated XPath node when an existing node is deprecated or obsolete. By embedding replacement information directly within the YANG model, this proposal enables programmatic inference of replacements and automated tooling for managing node replacements.


--- middle

# Introduction

YANG {{RFC7950}} is a data modeling language used to define the configuration and operational data of network devices. As network technologies evolve, some nodes within YANG models may become deprecated or obsolete. This document proposes the introduction of a new mechanism using a YANG extension to specify the updated XPath node when an existing node is deprecated or obsolete. By embedding replacement information directly within the YANG model, this proposal enables a programmatic inference of replacements and enables automated tooling for managing node replacements.

# Problem Statement

Currently, the only way to communicate replacement paths for deprecated nodes is through separate, unstructured documents like CSV files or emails. This approach suffers from several drawbacks:

- It is inefficient, requiring manual searching and correlation of information.
- It is error-prone, as the external document may not accurately reflect the node structure and paths defined in the YANG files.
- It hinders automation, as there is no standardized way for tools to programmatically identify and report replacement nodes.

# Solution

To address the challenges associated with deprecated XPath nodes in YANG models, we propose leveraging an extension-based approach rather than introducing a new core keyword to the YANG language. This approach follows the recommendations in {{RFC8407}} for extending YANG functionality. Specifically, we introduce a custom extension, `cisco-ext:replacement-info`, which provides metadata indicating the appropriate replacement for a deprecated or obsolete node. This extension will carry information such as the XPath or reference to the updated node, and optionally indicate whether the feature is scheduled for removal.

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

## `Cisco-IOS-XE-deprecation-regression-test-17131.yang` module

```yang
module Cisco-IOS-XE-deprecation-regression-test-17131 {
  yang-version 1.1;
  namespace "http://cisco.com/ns/yang/Cisco-IOS-XE-deprecation-regression-test";
  prefix depr-reg-test;

  import Cisco-IOS-XE-depr-reg-test-helper-17131 {
    prefix depr-reg-test-helper;
  }

  import cisco-extensions {
    prefix cisco-ext;
  }

  container deprecation-regression-test {
    container configurations {
      container config {

        // Case 1 : Leaf deprecated with a replacement in the same container.
        leaf case1 {
          type uint8;
          status deprecated;
          description
            "This leaf gives information about Case 1 - Leaf deprecated with a
             replacement in the same container, leaf case1 (DEPRECATED)";
          cisco-ext:replacement-info
            "REPLACEMENT_ABSOLUTE_PATH:/Cisco-IOS-XE-deprecation-regression-test-17131:deprecation-regression-test/configurations/config/case1-replacement";
        }
        leaf case1-replacement {
          type uint16;
          description
            "This leaf gives information about Case 1 - Leaf deprecated with a
             replacement in the same container, leaf case1-replacement (NEW)";
        }

        // Case 2 : Leaf deprecated with a replacement in a different container.
        leaf case2 {
          type uint8;
          status deprecated;
          description
            "This leaf gives information about Case 2 - Leaf deprecated with a
             replacement in a different container, leaf i (DEPRECATED)";
          cisco-ext:replacement-info
            "REPLACEMENT_ABSOLUTE_PATH:/Cisco-IOS-XE-deprecation-regression-test-17131:deprecation-regression-test/configurations/replacementContainerCase2/case2-replacement";
        }

        // Case 3 : Leaf deprecated with a replacement located in another YANG file(Cisco-IOS-XE-deprecation-regression-test-file-2.yang)
        leaf case3 {
          type uint8;
          status deprecated;
          description
            "This leaf gives information about Case 3 - Leaf deprecated with a
             replacement located in another YANG file, leaf case3 (DEPRECATED)";
          cisco-ext:replacement-info
            "REPLACEMENT_ABSOLUTE_PATH:/Cisco-IOS-XE-deprecation-regression-test-file-2-17131:deprecation-regression-test-2/configurations/config/case3-replacement";
        }

        // Case 4 : Leaf inside a list deprecated with a replacement in a different list.
        list listCase4 {
          key "key-case4";
          leaf key-case4 {
            type string;
          }
          leaf case4 {
            type uint8;
            status deprecated;
            description
              "This leaf gives information about Case 4 - Leaf inside a list
               deprecated with a replacement in a different list, leaf case4
               (DEPRECATED)";
            cisco-ext:replacement-info
              "REPLACEMENT_ABSOLUTE_PATH:/Cisco-IOS-XE-deprecation-regression-test-17131:deprecation-regression-test/configurations/config/replacementListCase4/case4-replacement";
          }
        }
        list replacementListCase4 {
          key "key-case4-replacement";
          leaf key-case4-replacement {
            type string;
          }
          leaf case4-replacement {
            type uint8;
            description
              "This leaf gives information about Case 4 - Leaf inside a list
               deprecated with a replacement in a different list, leaf
               case4-replacement (NEW)";
          }
        }

        // Case 5 : Leaf inside a grouping deprecated, replaced in the same grouping, and used across multiple containers.
        grouping groupCase5 {
          leaf case5 {
            type uint8;
            status deprecated;
            description
              "This leaf gives information about Case 5 - Leaf inside a grouping
               deprecated, replaced in the same grouping, and used across multiple
               containers, leaf case5 (DEPRECATED)";
            cisco-ext:replacement-info
              "REPLACEMENT_REL_PATH:/Cisco-IOS-XE-deprecation-regression-test-17131:groupCase5/case5-replacement";
          }
          leaf case5-replacement {
            type uint16;
            description
              "This leaf gives information about Case 5 - Leaf inside a grouping
               deprecated, replaced in the same grouping, and used across multiple
               containers, leaf case5-replacement (NEW)";
          }
        }
        container containerP1 {
          uses groupCase5;
          description
            "This container gives information about Case 5 - Leaf inside a
             grouping deprecated, replaced in the same grouping, and used across
             multiple containers, using containerP1 (NEW)";
        }
        container containerP2 {
          uses groupCase5;
          description
            "This container gives information about Case 5 - Leaf inside a
             grouping deprecated, replaced in the same grouping, and used across
             multiple containers, using containerP2 (NEW)";
        }

        // Case 6 : Leaf inside a grouping deprecated, replaced in the same grouping but imported through a different module and used in various modules.
        container containerP3 {
          uses depr-reg-test-helper:groupCase6;
          description
            "This container gives information about Case 6 - Leaf inside a
             grouping deprecated, replaced in the same grouping but imported
             through a different module and used in various modules, leaf inside
             containerP3 (NEW)";
        }
        container containerP4 {
          uses depr-reg-test-helper:groupCase6;
          description
            "This container gives information about Case 6 - Leaf inside a
             grouping deprecated, replaced in the same grouping but imported
             through a different module and used in various modules, leaf inside
             containerP4 (NEW)";
        }

        // Case 7 : Leaf deprecated, replaced by a different leaf with the same name in a different location.
        leaf case7 {
          type uint8;
          status deprecated;
          description
            "This leaf gives information about Case 7 - Leaf deprecated, replaced
             by a different leaf with the same name in a different location, leaf
             case7 (DEPRECATED)";
          cisco-ext:replacement-info
            "REPLACEMENT_ABSOLUTE_PATH:/Cisco-IOS-XE-deprecation-regression-test-17131:deprecation-regression-test/configurations/config/replacementContainerCase7/case7";
        }
        container replacementContainerCase7 {
          presence "true";
          description
            "This container gives information about Case 7 - Leaf deprecated,
             replaced by a different leaf with the same name in a different
             location, container replacementContainerCase7 (NEW)";
          leaf case7 {
            type uint8;
            description
              "This leaf gives information about Case 7 - Leaf deprecated,
               replaced by a different leaf with the same name in a different
               location, leaf case7 (NEW)";
          }
        }

        // Case 8 : Leaf-list deprecated, replaced by another leaf-list
        leaf-list case8 {
          type uint8;
          status deprecated;
          description
            "This leaf-list gives information about Case 8 - Leaf-list deprecated,
             replaced by another leaf-list, leaf-list case8 (DEPRECATED)";
          cisco-ext:replacement-info
            "REPLACEMENT_ABSOLUTE_PATH:/Cisco-IOS-XE-deprecation-regression-test-17131:deprecation-regression-test/configurations/config/case8-replacement";
        }
        leaf-list case8-replacement {
          type uint8;
          description
            "This leaf-list gives information about Case 8 - Leaf-list
             deprecated, replaced by another leaf-list, leaf-list
             case8-replacement (NEW)";
        }

        // Case 9 : Empty leaf deprecated, replaced by another empty leaf
        leaf case9 {
          type empty;
          status deprecated;
          description
            "This leaf gives information about Case 9 - Empty leaf deprecated,
             replaced by another empty leaf, leaf case9 (DEPRECATED)";
          cisco-ext:replacement-info
            "REPLACEMENT_ABSOLUTE_PATH:/Cisco-IOS-XE-deprecation-regression-test-17131:deprecation-regression-test/configurations/config/case9-replacement";
        }
        leaf case9-replacement {
          type empty;
          description
            "This leaf gives information about Case 9 - Empty leaf deprecated,
             replaced by another empty leaf, leaf case9-replacement (NEW)";
        }

        // Case 10 : A container is deprecated with a replacement container.
        container containerCase10 {
          leaf case10 {
            type uint8;
            status deprecated;
            description
              "This leaf gives information about Case 10 - A container is
               deprecated with a replacement container, leaf case10
               (DEPRECATED)";
            cisco-ext:replacement-info
              "REPLACEMENT_ABSOLUTE_PATH:/Cisco-IOS-XE-deprecation-regression-test-17131:deprecation-regression-test/configurations/config/replacementContainerCase10/case10-replacement";
          }
          status deprecated;
          description
            "This container gives information about Case 10 - A container is
             deprecated with a replacement container, containerCase10
             (DEPRECATED)";
          cisco-ext:replacement-info
            "REPLACEMENT_ABSOLUTE_PATH:/Cisco-IOS-XE-deprecation-regression-test-17131:deprecation-regression-test/configurations/config/replacementContainerCase10";
        }
        container replacementContainerCase10 {
          leaf case10-replacement {
            type uint8;
            description
              "This leaf gives information about Case 10 - A container is
               deprecated with a replacement container, leaf case10-replacement
               (NEW)";
          }
          description
            "This container gives information about Case 10 - A container is
             deprecated with a replacement container, replacementContainerCase10
             (NEW)";
        }

        // Case 11 : A multiple deprecated items are replaced by a single item.
        leaf case11a {
          type uint8;
          status deprecated;
          description
            "This leaf gives information about Case 11 - A multiple deprecated
             items are replaced by a single item, leaf case11a (DEPRECATED)";
          cisco-ext:replacement-info
            "REPLACEMENT_ABSOLUTE_PATH:/Cisco-IOS-XE-deprecation-regression-test-17131:deprecation-regression-test/configurations/config/case11-replacement";
        }
        leaf case11b {
          type uint8;
          status deprecated;
          description
            "This leaf gives information about Case 11 - A multiple deprecated
             items are replaced by a single item, leaf case11b (DEPRECATED)";
          cisco-ext:replacement-info
            "REPLACEMENT_ABSOLUTE_PATH:/Cisco-IOS-XE-deprecation-regression-test-17131:deprecation-regression-test/configurations/config/case11-replacement";
        }
        leaf case11-replacement {
          type uint16;
          description
            "This leaf gives information about Case 11 - A multiple deprecated
             items are replaced by a single item, leaf case11-replacement (NEW)";
        }

        // Case 12 : Leaf inside a list deprecated, but the replacement is in a different list type (not one-to-one mapping).
        list listCase12 {
          key "key-case12";
          leaf key-case12 {
            type string;
          }
          leaf case12 {
            type uint8;
            cisco-ext:replacement-info
              "REPLACEMENT_ABSOLUTE_PATH:/Cisco-IOS-XE-deprecation-regression-test-17131:deprecation-regression-test/configurations/config/replacementListCase12/case12-replacement";
          }
        }
        list replacementListCase12 {
          key "key-case12-replacement";
          leaf key-case12-replacement {
            type string;
          }
          leaf case12-replacement {
            type uint8;
          }
        }

        // Case 13 : In a choice-case, the leaf is deprecated in one case with an alternative provided in a new case.
        choice choice13 {
          case caseCase13 {
            leaf case13 {
              type uint8;
              status deprecated;
              description
                "This leaf gives information about Case 13 - In a choice-case, the
                 leaf is deprecated in one case with an alternative provided in a
                 new case, case13 (DEPRECATED)";
              cisco-ext:replacement-info
                "REPLACEMENT_ABSOLUTE_PATH:/Cisco-IOS-XE-deprecation-regression-test-17131:deprecation-regression-test/configurations/config/choice13/case13-replacement";
            }
          }
          case caseCase13Replacement {
            leaf case13-replacement {
              type uint8;
              description
                "This leaf gives information about Case 13 - In a choice-case, the
                 leaf is deprecated in one case with an alternative provided in a
                 new case, case13-replacement (NEW)";
            }
          }
        }

        // Case 14 : A single leaf is deprecated, and its functionality is distributed among multiple new leaves.
        leaf case11 {
          type uint16;
          status deprecated;
          description
            "This leaf gives information about Case 14 - A single leaf is
             deprecated, and its functionality is distributed among multiple new
             leaves, case11 (DEPRECATED)";
          cisco-ext:replacement-info
            "REPLACEMENT_ABSOLUTE_PATH:/Cisco-IOS-XE-deprecation-regression-test-17131:deprecation-regression-test/configurations/config/case14a-replacement,
             REPLACEMENT_ABSOLUTE_PATH:/Cisco-IOS-XE-deprecation-regression-test-17131:deprecation- regression-test/configurations/config/case14b-replacement";
        }
        leaf case14a-replacement {
          type uint8;
          description
            "This leaf gives information about Case 11 - A single leaf is
             deprecated, and its functionality is distributed among multiple new
             leaves, case14a-replacement (NEW)";
        }
        leaf case14b-replacement {
          type uint8;
          description
            "This leaf gives information about Case 11 - A single leaf is
             deprecated, and its functionality is distributed among multiple new
             leaves, case14b-replacement (NEW)";
        }

        // Case 15 : A leaf is deprecated without alternate as the feature is no longer supported in higer versions.
        leaf case15 {
          type uint8;
          status deprecated;
          description
            "This leaf gives information about Case 15 - A leaf is deprecated
             without alternate as the feature is no longer supported in higer
             versions, case15 (DEPRECATED)";
          cisco-ext:replacement-info "REPLACEMENT_ABSOLUTE_PATH:None";
        }

        // Case 16: An container is deprecated with a replacement, and the replacement value is specified at each level of the container and its child leaf elements.
        container containerCase16 {
          presence "true";
          status deprecated;
          description
            "This container gives information about Case 16 - An empty container is
             deprecated with a replacement, containerCase16 (DEPRECATED)";
          cisco-ext:replacement-info
            "REPLACEMENT_ABSOLUTE_PATH:/Cisco-IOS-XE-deprecation-regression-test-17131:deprecation-regression-test/configurations/config/replacementContainerCase16";

          leaf case16-leaf1 {
            type string;
            status deprecated;
            description
              "This leaf gives information about Case 16 - Leaf case16-leaf1 is
               deprecated within the deprecated container, case16-leaf1
               (DEPRECATED)";
            cisco-ext:replacement-info
              "REPLACEMENT_ABSOLUTE_PATH:/Cisco-IOS-XE-deprecation-regression-test-17131:deprecation-regression-test/configurations/config/replacementContainerCase16/case16a";
          }

          leaf case16-leaf2 {
            type uint8;
            status deprecated;
            description
              "This leaf gives information about Case 16 - Leaf case16-leaf2 is
               deprecated within the deprecated container, case16-leaf2
               (DEPRECATED)";
            cisco-ext:replacement-info
              "REPLACEMENT_ABSOLUTE_PATH:/Cisco-IOS-XE-deprecation-regression-test-17131:deprecation-regression-test/configurations/config/replacementContainerCase16/case16b";
          }
        }

        container replacementContainerCase16 {
          presence "true";
          description
            "This container gives information about Case 16 - Replacement for the
             deprecated containerCase16, replacementContainerCase16 (NEW)";

          leaf case16a {
            type string;
            description
              "This leaf gives information about Case 16 - Replacement for
               case16-leaf1, case16a (NEW)";
          }

          leaf case16b {
            type uint8;
            description
              "This leaf gives information about Case 16 - Replacement for
               case16-leaf2, case16b (NEW)";
          }
        }
      }
    }
  }
}
```

## `Cisco-IOS-XE-deprecation-regression-test-helper-module-17131.yang` module

```yang
module Cisco-IOS-XE-depr-reg-test-helper-17131 {
  yang-version 1.1;

  namespace "http://cisco.com/ns/yang/Cisco-IOS-XE-depr-reg-test-helper";

  prefix depr-reg-test-helper;

  organization "Cisco Systems, Inc.";

  contact "Cisco Systems, Inc.";

  import cisco-extensions {
    prefix cisco-ext;
  }

  // Case 6 : Leaf inside a grouping deprecated, replaced in the same grouping but imported through a different module and used in various modules.
  grouping groupCase6 {
    leaf case6 {
      type uint8;
      default 10;
      status deprecated;
      description
        "This leaf gives information about Case 6 - Leaf inside a grouping
         deprecated, replaced in the same grouping but imported through a
         different module and used in various modules, leaf case-6
         (DEPRECATED)";
      cisco-ext:replacement-info
        "REPLACEMENT_REL_PATH:/Cisco-IOS-XE-depr-reg-test-helper-module-17131/groupCase6/case6-replacement";
    }
    leaf case6-replacement {
      type uint8;
      default 11;
      description
        "This leaf gives information about Case 6 - Leaf inside a grouping
         deprecated, replaced in the same grouping but imported through a
         different module and used in various modules, leaf case6-replacement
         (NEW)";
    }
  }
}
```

## `Cisco-IOS-XE-deprecation-regression-test-file-2-17131.yang` module

```yang
module Cisco-IOS-XE-deprecation-regression-test-file-2-17131 {
  yang-version 1.1;
  namespace "http://cisco.com/ns/yang/Cisco-IOS-XE-deprecation-regression-test";
  prefix depr-reg-test-2;

  import cisco-extensions {
    prefix cisco-ext;
  }

  container deprecation-regression-test-2 {
    container configurations {
      container config {
        // Case 3 : Leaf deprecated with a replacement located in another YANG file(Cisco-IOS-XE-deprecation-regression- test-file-2.yang)
        leaf case3-replacement {
          type uint8;
          default 15;
        }
      }
    }
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

The authors acknowledge the original work on YANG Next Steps [YANG-NEXT] that provided initial inspiration for this proposal.

[YANG-NEXT] https://github.com/netmod-wg/yang-next/issues/130
