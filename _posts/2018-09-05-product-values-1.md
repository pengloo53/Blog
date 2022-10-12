---
title: 不要将责任推卸给用户 - 我的产品观
date: 2018-09-05 20:43
tags: [随笔,产品]
---

【1】

新员工小A入职，拿着公司配的办公笔记本，由于电脑接口不够用，想配一个蓝牙鼠标，但是蓝牙貌似不怎么好使，明明电脑是支持蓝牙的，但就是连不上鼠标，于是去找IT同事帮忙：

- 小A：您好，请问为什么电脑明明有蓝牙模块，但是却使用不了呢？
- IT：当然不能用啦，公司信息安全管控，不允许使用蓝牙。
- 小A：使用蓝牙，跟信息安全有什么关系？
- IT：因为蓝牙有传输文件的功能，信息安全禁止传文件的。
- 小A：哦，可是，我并不传输文件呀，连接鼠标也不能传文件呀
- IT：公司电脑是禁止使用蓝牙功能的，如果非要用蓝牙，需要你走审批开通蓝牙权限
- 小A：啊，哦，好吧，那算了

正常情况下，对话一般到这里就结束了，你觉得整个过程的的用户体验怎么样？IT岗位在许多传统公司应该属于服务部门，服务部门最主要的指标是服务满意度，假设你是上面的这位IT担当，你觉得你的服务让用户满意了吗？

在我看来，用户体验显然不够好。如果换作我是小A，我就想问：为什么不能只是禁止传输文件呢？我使用一个蓝牙鼠标也不能传输文件呀。

IT担当可能会说，这个问题我处理得也没错呀，由于技术限制，确实不能做到，只限制文件传输，而不限制蓝牙鼠标的使用。但是，**这跟用户有什么关系呢？**用户使用蓝牙鼠标违反公司信息安全制度了吗？用户凭什么为了你的所谓技术限制，而牺牲自己的用户体验呢？

对呀，这个过程用户有错吗？我是一个IT工程师，我理解也能体会，很多时候IT这块确实有很多令人无解的事情，也很受委屈。可是，我们作为服务部门给客户提供服务。**服务即是我们的产品**，在我看来，我们的产品没有带来好的用户体验，就是我们自己的问题。如果发现一些问题，是由于我们自己的技术限制，而导致无法解决，我们更应该去追寻更好的解决方案，而不是将责任推卸给用户。

【2】

> 不要将责任推卸给用户

这是二爷在他的产品课上提出的观点，我看到的时候，耳目一新。二爷在讲《验证码是个好设计吗》一课时，指出验证码不是一个好的设计，我听后感到很困惑，难道不是每个网站登陆都要有验证码的吗？为什么说它不是个好的设计？

验证码设计的初衷是为了防止机器扮成人，去占用原本为人准备的资源。比如，机器利用脚本程序不断模拟尝试登陆来破解密码。所以，验证码的设计，是为了区分，你是人还是机器，能通过验证的，证明你是人。然而随着人工智能图像识别越来越厉害，验证码也一直在进化升级，甚至到了，真人也通过不了的程度。还记得12306网站那个变态的验证码吧。

如果有用户因为验证码的问题，埋冤并指责我们的时候，也许这时，我们正在被网站攻击的问题搞得焦头烂额，但面对用户的抱怨，也只能说「对不起，是我们的问题，带来不便请谅解」，而不是让用户「走审批去吧」。这才是一个合格的IT担当（或产品经理）该有的态度。

我们可能觉得很委屈，网站被攻击了，也不是我们的问题，我们也是迫不得已才提高验证码的级别。是的，没错，但我们应该明白，**这更不是用户的问题**。如何区分前台是人还是机器，而防止被攻击，这本应该就是服务器端该做的事，但由于技术难题，服务器端无法解决这个问题，只能将它推卸给用户，设置各种障碍来让用户自己去证明。所以，验证码并不是一个好的设计。

【3】

在我们的工作中，存在着太多这样的例子，值得深思。尤其是传统企业的IT部门。因为我们面对的是企业内部的用户，企业在做信息化转型，我们肩负重任，给用户提供信息化服务。

然而现实情况下，我们总是推进得力不从心，系统上线了，却用不起来，反倒给用户造成困扰，增加工作量，用户一边线下处理着业务，一边难受的使用着系统。典型的为了信息化而信息化。为什么会存在这样的问题？

我不否认这里存在各种各样的问题，例如组织问题，管理问题，业务标准化问题等等，我们暂且抛开这些问题，单纯从产品的角度思考一下，我们的产品合格了吗？**我们作为系统的推广以及实际负责人，产品没有用起来，我们更应该找找自己的问题，要谨记「不要将责任推卸给用户」的这句话。**

多思考一下，用户不想用的根本原因是什么？从而去解决问题，而不是将责任推卸给用户，而去责怪他们不配合怎样怎样，用户为什么不配合呢？假设你的产品足够的优秀，用户又有什么理由不用呢？我们最大的问题就是没有危机感，总是站在自己的立场去思考问题，认为服务的对象是内部用户，就可以忽略用户体验，反正你用或是不用，系统都在那里，这种消极的对待方式本身就是不对的。**假如我们是一家向C端用户提供产品服务的公司，如果用户不买单，只能说明你的产品做得不好，让用户用的不爽，再多的理由也是白扯，面临只可能是倒闭**。

【4】

在很长一段时间里，我总是在思考一个问题：为什么传统企业信息化推进这么困难？是因为用户真的留恋传统作业方式吗？是他们不想改变的原因吗？站在IT的角度，我并不这样认为，如果一个产品真的做得好的话，用户应该自然而然会去拥簇。

这仅仅是我的一个产品观，也许想得过于理想，但至少要向着理想的方向前进，才会有所收获。好了，将「不要将责任推卸给用户」这句话，送给可能正在做产品的你，希望对你有所启发。