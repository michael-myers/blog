+++
date = "2017-02-13T23:41:37-05:00"
title = "Evaluating Equity as Compensation"
Tags = ["equity", "compensation", "business"]
Description = ""
Categories = ["business"]
+++

The traditional model of entrepreneurship is:

1. concept
2. plan
3. venture funding
4. exit

It seems like most people just unquestioningly accept that this is the natural sequence of events, but why is it this way? Isn't there something missing from this? Like what about the focus on the customer, or about actually making a great service or product? If a business is working, why do you need to leave? What's the rush? Why do you immediately go from borrowing money to looking for a buyer? This sounds more like house flipping than entrepreneurship. Accepting venture capital funding means accepting their short-sighted outsider influence, and this is what creates the pressure for an exit (rather than the creation of a stable, sustainable business). 

The frothy market of dollar-chasing VC-backed pump & dump schemes is probably the worst way to try to generate a world-changing technological breakthrough. *We live in a world where every problem left worth solving is a difficult multi-disciplinary one that requires long-term-horizon thinking*. If you are just planning to make money, you may not care. If you are paying some lip service to making the world a better place, though, this seems like a case of some misaligned values.

What about the technical staff, in all of this? The scientists, engineers, programmers. They get left holding the bag on step 4, that's what. They get [rolled](http://www.urbandictionary.com/define.php?term=rolled&defid=175634). At best, after "exit," they have a new management team and a 5-10% retention bonus for a couple of years. More likely, everything they liked about the job culture goes out the window and they get to watch the 2 or 3 founders drive off in their new luxury sport sedans, with a cash cushion that makes them set for life. At worst, the technical staff will get laid off shortly after the acquisition and/or the business will implode.

Most people over the age of 25 have become wise to this trajectory of events. For tech labor, it's theoretically still a seller's market (seller, here, being the *employee* in the employee-employer relationship). So prospective employees naturally want to be paid not just a salary, but to actually feel like they are sitting at the table. High-value employees want to be cut in (to use another poker metaphor). If you as the employer *don't* cut them in, science has shown: [you will be getting 5% less value out of them than you could be](http://esop.com/pdf/esopHistoryAndResearch/researchEvidence.pdf), if you can even hire and retain them in the first place.

Offers of equity to new employees are exceedingly common in venture-capital startup land. But even in government land, it's not unheard of to work for a company with strong employee-ownership values. SAIC [is (was, before its board took its stock public and then the model imploded) one well-known success story](http://www.beyster.com/SAIC-Solution-Singh-Review.pdf), and in the early 2000's I worked for another [lesser-known company](https://www.parsons.com/markets/Pages/Sparta.aspx) that used an identical compensation system (the founders of the two companies were friends, actually, and launched the companies in parallel). I have actually worked under a few different equity models over the years: a fully-employee-owned model (some in stock, some in NQO options), a profit-sharing "equity" model, and a no-equity model.

Business finance is not my specialty, but I believe that we as engineers should not adopt a learned helplessness. Many of us have a disinterest in the topic of money. Yes, equity compensation is complicated – even suspiciously so – and dry. Boring, even crass. But we should take an interest in ourselves, because nobody else will. Don't accept being exploited. This is about the empowerment of labor vs. capital.

[The Open Guide to Equity Compensation](https://github.com/jlevy/og-equity-compensation) is a community project to aggregate information to help improve the financial literacy of people in tech, specifically, to spread the kind of sophistication necessary to evaluate a compensation offer that is either all or partially made in terms of company equity. If you've never read this guide, do so now! There is seriously good info there, and you may save yourself from making an expensive mistake.

The Open Guide still has a gap when it comes to LLCs, though, so I will add some wisdom from personal experience. If you plan on working at a small less-established or newly established LLC, and are considering an equity offer:

- know that LLCs' equity sharing structure is different from Corporations, and advice on LLC equity is much harder to find, since LLCs are newer and less common than Corporations
- the equity offer should be in writing, *signed*
- the offer should specify the award *and* vesting timeline. If an *option* award, it should specify the *award* *price*
- the offer should specify the *valuation method*
- the offer should be in *percentage* terms, not in terms of some malleable *unit* (fraction with an unknown denominator)
- the offer should contain no loophole words like "intend to," but be worded as a commitment, like "shall" or "will"
- if the equity-sharing plan structure is still TBD, *consider the offer worthless*

Especially that last bullet, since it supercedes everything else. Once you accept the offer of employment, you lose all leverage. Once you have lost leverage, your "equity" (if you ever get it) will be defined as something worth nearly zero, in a way that most favors the company. Your *award* of, say, a defined percent of the valuation of the company can be interpreted to mean an *option* grant to *purchase* some equity. Remember, it's not an *award* if you have to *buy* it, it has no present value if you have to buy it at-or-above the present value when granted, and it's not *worth anything* if you can't sell it. Check: are you basically buying an illiquid asset with a tax consequence? Is this thing going to pay dividends until you can sell it? And for an LLC, also realize that you *might* have to give up employee (W-2) status and start paying more taxes on your salary, too.

See, with LLCs, you cannot get stocks, you can only get "membership interest units." In an LLC, equity might be in the form of Profits Interest Units (not to be confused with *profit sharing*). "Profit interest" is a junior form of equity in every sense of the term – if you want the Big Boy equity (ownership) in an LLC you want "Capital Interest." Either way though, unlike a stock, these two kinds of LLC Interests may be restricted in a way that prevents them from being freely transferred. You might have to hold it until the company itself is sold. And there are complicated consequences for taxation and accounting, that all threaten to make them more trouble than they are really worth, especially for small amounts.

Here is a table to help roughly compare LLC equity offers with traditional coporation equity offers.

| Equity in a C- or S-Corporation          | Equity in a LLC (the closest parallel)   |
| :--------------------------------------- | :--------------------------------------- |
| Stock Option                             | Profits Interests Units *or* Options for Capital Interests |
| Restricted Stock Units (RSU)             | Capital Interests Units                  |
| "Phantom Stock"                          | (Membership Interest) Unit Rights, a.k.a. "Phantom Equity" |
| Stock Appreciation Rights (SAR), a.k.a. Phantom Stock Option | (Membership Interest) Unit Appreciation Rights |

**The more established an LLC company is, the less attractive a Profits Interest Unit becomes, relative to a Capital Interest Unit.** That is because a Profits Interest value is only in the *future appreciation* of the value of the company, whereas a grant of Capital Interest is an immediate share of the value of the LLC as of the date that the interest is granted. In addition, the longer an employer can drag its feet and delay the award of a Profit Interest Unit, the less value it is to you. That's why award date, strike price, and vesting schedule are all very important.

If Amazon were an LLC, would you rather be granted – today – a 1% [*profit interest*](https://ycharts.com/companies/AMZN/profit_margin)? Or 1% [*equity*](https://ycharts.com/companies/AMZN/market_cap)? Amazon is infamously unprofitable, but also enormous. And if it were a profit interest, would you rather have that five years ago, or today? Five years ago obviously: even though Amazon is unprofitable, a PIU represents both profit and appreciation of company valuation, which for Amazon is like a 450% increase.

Also, keep in mind that an LLC can always allocate its gross profit in any given year so as to lower (or eliminate) net profit, disbursing the money in (almost) any way that it sees fit. The management team feels like paying all the profits out as bonuses this year? That comes at the expense of the PIU holders. A company car for the CEO? Sorry PIU holders. If you get a great PIU award and it's fully vested today, but tomorrow the LLC is sold? Sorry, there hasn't been any growth in the company value over that period, so your PIUs are worth zero.

Anyway, I hope this has been useful. If you disagree or think I am misinformed, I am open to discussions; find me on Twitter. I encourage everyone to educate themselves using multiple sources, and I don't claim to be an authority on this subject. In fact, here's a disclaimer.

**Disclaimer**

*This blog post and all associated comments and discussion do not constitute legal or tax advice in any respect. *The author has prepared this material for informational purposes only, and it is not intended to provide, and should not be relied on for, tax, legal or accounting advice. The author is not a licensed practitioner in taxes, law, or accounting. No reader should act or refrain from acting on the basis of any information presented herein without seeking the advice of counsel in the relevant jurisdiction. The author(s) expressly disclaim all liability in respect of any actions taken or not taken based on any contents of this guide or associated content.*