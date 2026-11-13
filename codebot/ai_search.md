# Supporting AI Search

Load this card when a task makes a page searchable by AI Navigation (AI
Search): a `GoTo` action control, an `AINavigationPrompt`, or the links and
chooser actions a search selects from, with their documentation. Its rules are
generated from the
[AI Navigation contract](../reference/protocols.md#ai-navigation-and-ai-search),
which owns them. The searchable destinations are ordinary [links](links.md)
and [actions](actions.md); their meaning comes from the
[source documentation](../reference/documentation.md#domain-references-and-discoverability).

## Order of work

1. Plan each supported goal as a reachable destination and expose it through
   links or finite chooser actions (`ai.obligations.planning`,
   `ai.candidates.*`).
2. Opt the page in with a `GoTo` control or a prompt resource
   (`ai.search.*`, `ai.prompt.*`).
3. Document each destination's meaning and domain vocabulary
   (`ai.obligations.*`), then compile with Studio and fix every reported
   rule.

<!-- generated:codebot-rules task=ai-search -->
<!-- Generated from the reference rule markers; edit the reference and regenerate. -->

## Rules to apply

Studio does not check these rules. Each is quoted from its owner.

### AI Navigation and AI Search

Owner: [protocols.md#ai-navigation-and-ai-search](../reference/protocols.md#ai-navigation-and-ai-search)

- `ai.search.definition`: **AI Navigation** and **AI Search** name the same capability: use a person’s text or editable voice transcript to select an existing eligible `GoTo` choice. The App supplies destination meaning; the client constructs and validates complete requests before selection; AI returns only candidate identifiers. A candidate is one offered invocation or finite chooser leaf. Two eligible leaves of one Go to action are distinct candidates.
- `ai.search.availability`: This feature requires an implementing client and a supporting modern Hub. It is unavailable on legacy Hubs and on non-task pages. On a task page it is shown when the task's Go to control is shown with eligible choices, the Hub supports it, an AI provider is available to the client (the Hub's provider or the person's own configured account), and Hub policy does not disable it, whatever the page currently holds: it never depends on the page's Request, Response, pending operation or prompt. Its candidates are exactly the task's `GoTo` choices, the same currently eligible set the Go to control offers, so AI Search and Go to always agree; nothing outside that set is a candidate. If choices become unavailable after the composer opens, it reports the condition inside the composer on submission and keeps the query. If none matches, it reports no match and keeps the query. App guidance (a Union or page prompt) is optional and only improves interpretation. Search and task-scoped Assistant are independent features.
- `ai.search.prompt-no-authority`: A navigation prompt does not grant access or enable tool execution.

### Prompt attribute and composition

Owner: [protocols.md#prompt-attribute-and-composition](../reference/protocols.md#prompt-attribute-and-composition)

- `ai.prompt.editor-page`: A request editor uses its own Request variant's page declaration; an instruction that opens that editor does not become its page. A synthetic request-list landing has only the Union declaration available.
- `ai.prompt.composition`: Resolve the Union and current page resources through ordinary localization, in that order, and concatenate their texts with a blank line. Both declarations contribute. Page guidance adds local meaning and can refine common selection preferences; it does not erase the Union guidance. An absent declaration contributes nothing. Guidance never establishes availability or adds candidates. An offered `GoTo` control needs neither declaration; the platform uses its ordinary destination labels, descriptions and current eligible choices. A control with no eligible choices does not establish availability. Detect the case-sensitive control name `GoTo`, independently of its localized or template caption and placement.
- `ai.prompt.no-runtime-arguments`: The App authors the resource at development time; a navigation prompt takes no runtime message arguments. Runtime page and item values are supplied separately as data. The platform supplies the general selection instructions and result format.
- `ai.prompt.cannot-expand`: App guidance and runtime text cannot expand the candidate set or change construction, access, validation, confirmation, or submission rules. The existing `LLMToolCall` attribute continues to control Hub tool projection; it is not this page opt-in.

### Candidate discovery and assembly before AI

Owner: [protocols.md#candidate-discovery-and-assembly-before-ai](../reference/protocols.md#candidate-discovery-and-assembly-before-ai)

- `ai.candidates.app-wide-coverage` (judgment): App-wide coverage requires the App to expose the intended destinations through Go to choices or their finite chooser leaves from the page offering that scope. The client does not traverse unoffered pages or combine raw enum values into speculative links.

### App and CodeBot authoring obligations

Owner: [protocols.md#app-and-codebot-authoring-obligations](../reference/protocols.md#app-and-codebot-authoring-obligations)

- `ai.obligations.planning` (judgment):

  | Area | Obligation |
  | --- | --- |
  | Planning | Associate each supported user goal with an ordinary reachable destination. Expose the Go to choices or eligible finite chooser leaves needed for the advertised search scope and make their requests constructible from real available data. A navigation request may open an editor where subsequent user input occurs. |

- `ai.obligations.comments` (judgment):

  | Area | Obligation |
  | --- | --- |
  | Comments | Describe each destination's purpose, scope, consequences, and distinguishing domain vocabulary in extracted source documentation. Document topic choices as well as the generic request. |

- `ai.obligations.attributes` (judgment):

  | Area | Obligation |
  | --- | --- |
  | Attributes and data | Use NextRequest, applicable ConstrainedBy, GoTo ActionControl membership, and request-owned construction/confirmation declarations. Supply their actual runtime data; the AI does not create allowed sets. |

- `ai.obligations.prompts` (judgment):

  | Area | Obligation |
  | --- | --- |
  | Prompt resources | Guidance is optional for an offered GoTo control. When needed, put shared interpretation in the Union resource and local interpretation in the page resource. Keep cross-topic guidance here and destination-specific meaning with its documented destination, with one owner for each fact. |

- `ai.obligations.review` (judgment):

  | Area | Obligation |
  | --- | --- |
  | Review | Verify representative formal references, abbreviations, and everyday descriptions against the context the client will actually supply. Check unsupported and ambiguous queries as well as successful selections. |

- `ai.obligations.private-comments` (judgment): Ordinary private code comments are not extracted navigation knowledge. Missing domain meaning must be authored; collecting more schema does not create it.
- `ai.obligations.domain-references` (judgment): For the Tax App, statutory references, form/schedule numbers, formal names, abbreviations, and common names must all be useful ways to find supported topics. A description can connect a qualified-business-income destination to Section 199A, Form 8995/8995-A, and QBI. These associations identify a topic; they do not assert that a taxpayer qualifies for a deduction. The actual destination, jurisdiction, tax year, and supported scope come from the App. When several topics share a form or citation, supply enough context to select the appropriate topic or its available overview, or offer the relevant alternatives for the person to choose. Multiple OpenSection choices must carry their individual topic meaning.

## Rules Studio enforces

Studio checks these rules and reports errors or warnings as their owners specify. The owner holds the full rule.

### Prompt attribute and composition

Owner: [protocols.md#prompt-attribute-and-composition](../reference/protocols.md#prompt-attribute-and-composition)

- `ai.prompt.declaration`: AINavigationPrompt names one key in the declaring package StringResources
- `ai.prompt.placement`: AINavigationPrompt goes on the protocol Union or a page-defining variant, at most once per location
- `ai.prompt.studio`: Studio validates prompt placement, a nonempty key and an existing nonempty StringResource

<!-- /generated -->
