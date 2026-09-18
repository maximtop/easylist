# Run instruction: EasyList with uBlock Origin in Firefox

This file is the run instruction for the EasyList repository. The run's blocker is uBlock Origin
(uBO) in Firefox, force-installed from the current signed release through Firefox enterprise
policies. Every step below is the run's obligation — perform it exactly as written: the host reads
the blocker state back itself and credits a phase only when that state holds exactly what this
instruction declares.

The run loads its filter guidance at start from these role documents:

- [uBlock Origin static filter syntax](https://github.com/gorhill/uBlock/wiki/Static-filter-syntax)
- [EasyList contributing guide](https://github.com/easylist/easylist/blob/master/CONTRIBUTING.md)

## Preparation

The run image provides `curl`, `jq`, `node`, `git` and `unzip` for these steps; there is no
`python`, `perl` or `wget`.

launch: firefox

uBlock Origin is installed into Firefox as a signed XPI, force-installed through enterprise
policies; the host writes the policies file itself before every browser start, so do not write one.
Your job is to fetch the XPI and declare how the host must install it.

1. Download the current signed Firefox build of uBlock Origin — the current release version only,
   never a frozen or pinned tag. Ask the public GitHub releases API
   `https://api.github.com/repos/gorhill/uBlock/releases/latest` and take the asset whose name
   ends with `.firefox.signed.xpi`; save it inside the run workspace.
2. Assert the saved XPI exists in the run workspace and is not empty.
3. Finish with the Firefox launch declaration in your terminal payload:
    - `launchFamily`: `firefox`.
    - `extensionId`: `uBlock0@raymondhill.net`, uBO's published Firefox id.
    - `xpiPath`: the saved XPI, as a path relative to the working directory.
    - `managedStorage`: the managed-storage document uBO reads from `browser.storage.managed`, as
      JSON text — exactly the document below: EasyList as uBO ships it plus `user-filters`.
      Without `user-filters` in the selection uBO never applies the candidate rule at all. This
      selection is the run's executable baseline: it is what every phase runs with and what the
      run report names.
    - `userFiltersKeyPath`: `["adminSettings", "userFilters"]` — the key inside that document the
      host fills with the exact contents of the user-filters file named under State verification.
      Do not create or write that file yourself: the host creates and maintains it. Nothing here
      needs the XPI unpacked or inspected; once it is saved and checked, finish.

The managed-storage document, verbatim:

```json
{
    "adminSettings": {
        "selectedFilterLists": [
            "user-filters",
            "easylist"
        ]
    }
}
```

`easylist` is uBO's stock EasyList subscription, the list this repository publishes; the run's
baseline is therefore the published list alone, which is what a reporter running uBO with the
default selection sees.

## Rule application

The host maintains the user-filters file `filters-agent/ublock/user-filters.txt` itself; there are
no steps for a session to perform. Between phases the host writes that file — empty for the
baseline goal, exactly the candidate rule as one line for the candidate goal — rebuilds the
enterprise policies with the file's exact contents at the declared key path, relaunches the browser
so the force-installed uBO reads the regenerated managed storage at startup, and then reads the
file back. uBO consumes managed storage while Firefox applies the policies, never from a running
session's settings UI, which is why the relaunch is part of the application and not an extra step.

## State verification

After the application steps the host reads the blocker state back itself; it never accepts the
session's own report. The host's declaration:

read: managed-storage-file filters-agent/ublock/user-filters.txt

The target is relative to the run's checkout root; the host resolves it there. The file is the one
the host maintains and the one whose contents the enterprise policies carry into uBO's managed
storage — one rule per line, the candidate rule alone for a candidate phase.

The empty file credits the baseline phase and the exact candidate line credits the candidate phase.
The run's three phases are the ones every validation uses: Firefox with no extension, Firefox with
uBO and the list declared above on an empty user-filters file, and the same plus exactly the
candidate rule. The file read-back cannot see `selectedFilterLists`, so the phase proof reports the
enabled set from the declaration above — the lists Firefox applied when it force-installed the XPI —
and records that the user-filter state itself was credited from the file's content alone.

## Placement

Nothing is declared here on purpose: the run reads the checkout and finds the file and the line
itself. EasyList files rules by kind — site-specific hiding in
`easylist/easylist_specific_hide.txt`, site-specific blocking in
`easylist/easylist_specific_block.txt`, ad servers in `easylist/easylist_adservers.txt` — and keeps
every list in ASCII order with `fop`, so the run picks the list holding rules of the candidate's
shape and proposes the rule at its sorted position there. The user-filters file named under State
verification is the in-browser application path only; it never receives the proposed rule.

## Issue selection

Take issues that report ads or ad placeholders on real pages that EasyList should block. Work one
issue whose fix is a filter rule; skip tracking reports, cookie notices, feature requests,
infrastructure or meta tickets, and anything fixable only by changing the blocker itself rather
than the list.

- max-age-days: 30

## Report template

### Outcome

{{outcome}}

{{outcomeReason}}

{{versionUpdateHint}}

### Reproduced symptom

{{symptom}}

### Rule

{{rule}}

### Candidate for review

{{candidateForReview}}

### Still visible after the rule

{{stillVisible}}

### Executor and version

{{executor}} {{executorVersion}}

uBlock Origin in Firefox, installed from the signed release XPI named under Preparation, with
EasyList as the only list.

### Policy rationale

{{policyRationale}}

### Place in the list

{{listPlace}}

The file above is the list the run found for this kind of rule, and the proposed line is its
sorted position there. The user-filters file the run verified against is the in-browser
application path only.

### Missing information

{{missingInformation}}

### Artifacts

{{artifactsLink}}
