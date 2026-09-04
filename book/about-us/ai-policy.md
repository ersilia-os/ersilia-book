---
description: >-
  This policy has been developed with the support of the Non Profit AI Policy
  Builder from Fast Forward
---

# AI Policy

### Purpose of the Policy

Ersilia builds open-source AI tools for drug discovery, runs its own scientific research, and delivers training and capacity building with research institutions in the Global South. An AI policy is essential to ensure that artificial intelligence tools are developed and deployed responsibly, ethically, and in alignment with organizational values. This policy establishes clear guidelines for the use of AI to protect stakeholders, maintain trust, and promote equitable outcomes.

### AI Use

#### What we build

Ersilia develops and distributes AI/ML models for drug discovery, with a focus on infectious and neglected diseases. These are narrow, task-specific models: they predict properties of chemical compounds — bioactivity against a pathogen, toxicity, physicochemical characteristics — or generate candidate molecules. Researchers use them to prioritise which compounds are worth testing experimentally, reducing the time and cost of early-stage discovery. We process scientific data (molecular structures, disease data, experimental results) and limited personal information (names, contact details) to advance scientific discovery and improve health outcomes in resource-limited settings.

#### Claude by Anthropic

We have recently incorporated Claude by Anthropic to our work across three levels of the organisation: technical, to develop and maintain the Ersilia Model Hub; scientific, to support our research; and organisational, in fundraising, communications and administration.

Individuals working or collaborating at Ersilia are expected to exercise judgment about when and how to use it, based on the nature of the work and the risks involved, and to use the best tool available for the job.

### Interactions with Beneficiaries

AI tools are deployed directly to beneficiaries who use them for research and healthcare decision-making. These interactions require careful attention to usability, accessibility, and the provision of clear guidance on how to interpret AI-generated outputs. Users receive training to understand the capabilities and limitations of the AI tools to ensure informed and appropriate use in their work.

Our models support scientific decisions and every prediction is accompanied by a confidence estimate, based on how far a compound sits from the training distribution, so that users can judge how much weight to place on a result.

#### Collaborative work and training activities

Where AI is used in collaborative work with partners and in our training programs, the same principles apply, and partners retain ownership of their data and results.&#x20;

Our flagship capacity building program, the AI Incubator, has its own [AI Policy ](https://app.gitbook.com/s/HHO5jOS38euA5IqOgq6G/resources/ai-policy) and [intellectual property](https://app.gitbook.com/s/HHO5jOS38euA5IqOgq6G/resources/intellectual-property) property sections related to the use of AI during the program and the intelectual property of the projects data and outcomes.

### Data Collection Practices

Explicit consent is obtained from all users before collecting or processing their data, and clear information is provided about what data is collected and how it is used (see our [Privacy Notice](ersilia-privacy-notice.md)).&#x20;

#### Partner data and third-party AI tools

Partner data shared with Ersilia under confidentiality or intellectual property restrictions is not entered into third-party AI tools unless our agreement with that partner explicitly permits it. Where there is any doubt, we ask the partner before proceeding.

Our work is preclinical. We do not handle patient-level or person-identifiable information in any AI tool.

Where any of us reviews grant applications or manuscripts on behalf of funders or journals, we do not enter that content into AI tools. This is a binding condition of most funders and publishers we work with, and we treat it as absolute. This applies also to our training programs.

### Ethical Risks and Concerns

We identify four principal concerns: protecting research data privacy; ensuring model transparency; achieving equitable global access so that underserved communities are not excluded; and maintaining human judgment as AI becomes more capable. On the last point, the risk is that AI produces plausible outputs that go unchecked. The level of testing, validation and human review must match the stakes of the work: high scrutiny applies to models published in the Ersilia Model Hub, scientific conclusions and publications, work delivered to partner institutions, and anything shared with funders.

### Accountability for AI Decisions

Shared accountability is maintained between Ersilia staff who develop and maintain the AI models and the researchers or healthcare workers who use the AI outputs in their decision-making processes. Ersilia staff are responsible for ensuring that AI models are accurate, reliable, and appropriately documented, while end users are responsible for applying AI-generated insights appropriately within their professional contexts. This collaborative accountability framework recognizes that both developers and users play critical roles in ensuring ethical and effective AI deployment.

### Bias Detection and Prevention

AI models are tested across diverse datasets to ensure broad applicability. While third-party tools are used with the assumption that they are unbiased, ongoing vigilance is required to monitor for unexpected biases that may emerge in practice.

Some diseases, pathogens and genetic backgrounds are heavily underrepresented. We mitigate this in three ways: by auditing datasets before modelling and assessing whether a given dataset can support a reliable model at all; by training on large datasets and fine-tuning for our cases of interest; and by attaching confidence estimates to predictions so that users know how far a compound sits from the training distribution.

We also correct for the limits of public data by learning directly from the researchers we work with about conditions in the field.

Model performance is assessed through cross-validation and benchmarking and, where possible, through experimental validation with partners, measuring improvement in hit rates against conventional approaches. In some cases validation is conducted at sites independent from where the training data originated.

### Transparency in AI Decisions

Clear documentation is provided explaining how AI models work, their limitations, and the appropriate contexts for their use. AI models are made open source whenever possible, allowing users to inspect, understand and validate the underlying algorithms and methodologies. Models are published through the Ersilia Model Hub with their metadata, intended use and provenance documented, following [FAIR principles](https://www.go-fair.org/fair-principles/). Our development history is public on GitHub in real time.

We also publish our board minutes and report our income and expenses openly.

#### Disclosure of Claude use

* In the Ersilia Model Hub: where Claude has been used to incorporate, package or maintain a model, we state so explicitly in that model's documentation. A user of the Hub can therefore see how each model came to be there.
* In code: where Claude has assisted in developing a contribution, we record it in the commit metadata using an `Assisted-by:` trailer.
* In publications: see [_Scientific Publication section_](ai-policy.md#scientific-publication) below.
* In communication: we use Claude to help prepare content such as LinkedIn posts and our monthly newsletter, working from skills we have built and published for this purpose. The material is always drafted from information we provide, reviewed and edited by the person responsible before publication. We do not use Claude to speak for us in community discussion — in issues, pull requests or direct exchanges with collaborators — except for translation or language editing.

### Community Feedback and Reporting

Formal partnerships with organizations in the Global South provide ongoing feedback on the effectiveness and appropriateness of AI tools in diverse contexts. Users can report problems through our GitHub repositories and our community channels such as slack, and our partner institutions give us direct feedback on how the tools perform in their own research settings.

Consistent with our open-source commitment, the tools we build to work with Claude — including our skills and integrations — are released publicly under open licences for anyone to adopt or adapt. We test and learn from tools built by other organisations in turn.

### Third-Party AI Tools and Accountability

The principal third-party AI assistant we use is Claude, developed by Anthropic. Third-party AI tools are used in accordance with vendor terms of service, and we monitor their alignment with our organizational values.

Our use of these tools is governed by the following expectations:

* AI should be used where it meaningfully improves quality, efficiency, insight or capability, or reduces cost.
* Verification, testing and human judgment are required, at a level proportional to the stakes of the work.
* Responsibility for any output rests with the person who signs it off.
* Our team and our collaborators work in English as a second or third language. Translation and language editing enable participation.
* We protect confidential information as stablished in the [_Data Collection Practices_](ai-policy.md#data-collection-practices) above.

### Staff Training and Awareness

Staff members rely on their existing knowledge and expertise regarding AI ethics and best practices, with the expectation that they maintain awareness of evolving standards in the field. While formal training programs are not currently in place, staff are encouraged to engage in professional development opportunities and to share knowledge within the organization. As AI technologies and ethical considerations evolve, the organization may develop more structured training initiatives to ensure consistent understanding of responsible AI use across all team members.

### Scientific Publication

AI systems are not listed as authors on Ersilia publications. This position is shared without exception by[ ICMJE](https://www.icmje.org/), [Nature Portfolio](https://www.nature.com/nature-portfolio), [Science](https://www.science.org/content/page/science-journals-editorial-policies), [Elsevier](https://www.elsevier.com/about/policies-and-standards/generative-ai-policies-for-journals) and [PLOS](https://journals.plos.org/plosone/s/ethical-publishing-practice): an AI system cannot take responsibility for a piece of work, and authorship is inseparable from responsibility.

Where AI has been used substantively in preparing a publication, we disclose which tool was used, how it was used, and how the authors validated the output — following the specific requirements of the target journal. Language editing alone generally does not require disclosure, but we check the journal's rule.

### Contributions to Our Code

These expectations apply to everyone contributing to Ersilia repositories, including our own team.

* Responsibility: Ersilia staff and collaborators are responsible for any code they submit, regardless of how it was produced. They must understand the code and be able to explain it during review.&#x20;
* Disclosure: Where AI has assisted, record it using an `Assisted-by:` trailer in the commit metadata.&#x20;
* Autonomous agents: AI agents do not open pull requests autonomously in Ersilia repositories. A person reviews the code, understands it, and submits it under their own name.
* Licensing: The Ersilia Model Hub is released under [GPLv3](https://choosealicense.com/licenses/gpl-3.0/); other Ersilia repositories may use different open-source licences, and contributions must be compatible with the licence of the repository they are submitted to. Where a model is packaged from existing published work, its original licence is recorded and respected. We reserve the right to reject any contribution, AI-assisted or not, where provenance or copyright is in question.

### Environmental Footprint

We prioritise re-using existing models over training new ones, since inference is far less costly than training. Most of the models in the Ersilia Model Hub are packaged from existing published work. Our models are domain-specific, do not require GPUs and run on conventional hardware — a design decision driven by our mission, which also keeps our footprint small. We do not yet track our footprint systematically; as the repository grows we intend to implement tracking strategies.

### Regulatory Context and Review

Ersilia is a registered Spanish foundation and operates under EU law.

As a user of third-party AI systems, Ersilia is a deployer within the meaning of [Regulation (EU) 2024/1689](https://eur-lex.europa.eu/eli/reg/2024/1689/oj) (the EU AI Act), and takes measures to support AI literacy among its staff in accordance with [Article 4](https://artificialintelligenceact.eu/article/4/). We handle personal data in accordance with the [General Data Protection Regulation](https://eur-lex.europa.eu/eli/reg/2016/679/oj); our [privacy notice](https://ersilia.gitbook.io/ersilia-book/about-us/ersilia-privacy-notice) sets out what we collect and why.

The models we develop and release under free and open-source licences are narrow and task-specific: they predict properties of chemical compounds for use in preclinical research. They are not developed or released for clinical purposes — diagnosis, treatment decisions, or as components of medical devices — and we assess the regulatory requirements applicable to any new development on a case-by-case basis.

We align our practice with the [ERA Living Guidelines on the responsible use of generative AI in research](https://research-and-innovation.ec.europa.eu/news/all-research-and-innovation-news/updated-era-living-guidelines-responsible-use-generative-ai-research-2026-05-08_en), and with the principles of the African Union [Continental AI Strategy](https://au.int/en/documents/20240809/continental-artificial-intelligence-strategy) and Data Policy Framework — in particular the principle that AI developed for African contexts should be shaped by African researchers and institutions, which is the basis on which we work.

This policy is reviewed at least annually, and whenever we adopt a materially new AI capability.



_Current version: September 2026._

<br>
